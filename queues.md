# Queues

- [介绍](#introduction)
- [实现任务类](#writing-job-classes)
    - [生产任务类](#generating-job-classes)
    - [任务类的结构](#job-class-structure)
- [推送任务到队列](#pushing-jobs-onto-the-queue)
    - [延时任务](#delayed-jobs)
    - [从请求分发任务](#dispatching-jobs-from-requests)
- [运行队列监听器](#running-the-queue-listener)
    - [监管配置](#supervisor-configuration)
    - [队列监听器守护进程](#daemon-queue-listener)
    - [用队列监听器守护进程部署](#deploying-with-daemon-queue-listeners)
- [处理失败的任务](#dealing-with-failed-jobs)
    - [失败任务事件](#failed-job-events)
    - [重试失败任务](#retrying-failed-jobs)

<a name="introduction"></a>
## 介绍

Laravel 队列服务提供统一的API集成了许多不同的后端队列。队列允许你延后执行一个耗时的任务，例如延迟到指定时间发送邮件，这样大幅地加快了应用程序处理请求的速度。

<a name="configuration"></a>
### 配置

队列的配置文件保存在 `config/queue.php`。在这个文件中你可以找到框架中所有队列驱动的连接设置，包括数据库、[Beanstalkd](http://kr.github.com/beanstalkd)、[IronMQ](http://iron.io)、[Amazon SQS](http://aws.amazon.com/sqs)、[Redis](http://redis.io) 和一个同步驱动（本地使用）。

还包括一个驱动 `null` ，其仅仅是简单地舍弃队列任务。

### 驱动的必备条件

#### 数据库

为了能够使用 `database` 驱动，你需要建立一个数据库来保存任务。可以执行 `queue:table` Artisan指令创建一个迁移来建立这个数据库。一旦创建迁移，你可以用下面的 `migrate` 命令来迁移你的数据库

    php artisan queue:table

    php artisan migrate

#### 其他队列的依赖

下面的依赖是使用对应的队列驱动所需的扩展包：

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- IronMQ: `iron-io/iron_mq ~2.0`
- Redis: `predis/predis ~1.0`

<a name="writing-job-classes"></a>
## 实现任务类

<a name="generating-job-classes"></a>
### 生成任务类

默认情况下，你的应用中所有可以进入队列的任务都存放在目录 `app/Jobs` 下面。你可以使用 Artisan CLI 生成一个新的排队任务：

    php artisan make:job SendReminderEmail --queued

这个命令会在 `app/Jobs` 目录下面生产一个新的类。这个类会实现接口 `Illuminate\Contracts\Queue\ShouldQueue`，指示 Laravel 该任务应该推送到队列，而不是同步执行。

<a name="job-class-structure"></a>
### 任务类的结构

任务类非常简单，一搬只包含一个 ｀handle｀ 方法。队列处理任务的时候会调用这个方法。让我们看一下一个任务类的例子：

    <?php

    namespace App\Jobs;

    use App\User;
    use App\Jobs\Job;
    use Illuminate\Contracts\Mail\Mailer;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Bus\SelfHandling;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendReminderEmail extends Job implements SelfHandling, ShouldQueue
    {
        use InteractsWithQueue, SerializesModels;

        protected $user;

        /**
         * Create a new job instance.
         *
         * @param  User  $user
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Execute the job.
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function handle(Mailer $mailer)
        {
            $mailer->send('emails.reminder', ['user' => $this->user], function ($m) {
                //
            });

            $this->user->reminders()->create(...);
        }
    }

在这个例子中可以看到，我们直接把一个 [Eloquent model](/docs/{{version}}/eloquent) 传入一个任务的构造器中。因为任务使用了 `SerializesModels` trait， Eloquent 会在任务处理过程中被优雅的序列化和反序列化。如果你的排队任务在其构造器中包含了一个 Eloquent 模型，仅仅是模型的标识符被序列化到队列。在真正处理任务的时候，队列系统会自动地从数据库中取回整改模型实例整个过程对于你的应用是透明的，而且处理所有在序列化整个 Eloquent 模型实例过程中可能产生的问题。

`handle` 方法在队列处理任务的时候被调用。我们可以在任务的 `handle` 方法中提示需要的依赖。Laravel ［服务容器］(/docs/{{version}}/container) 会自动注入这些依赖。

#### 处理问题

如果任务在被处理的时候抛出异常，任务会被自动释放回到队列中以便再次尝试执行该任务。任务会一直被释放回队列直到达到应用程序的尝试上限。这个上限值可以通过 Artisan 命令 `queue:listen` or `queue:work`的的  `--tries` 开关设置。运行队列监听器的更多信息 [可以在下文找到](#running-the-queue-listener)。

#### 手动释放任务

如果你想手动 `release` 一个任务，生产的任务类中包含了一个 `InteractsWithQueue` trait，提供入口进入队列任务的 `release` 方法。这个 `release` 方法接收一个参数：秒数，你希望等待的时间直到任务再次获取。

    public function handle(Mailer $mailer)
    {
        if (condition) {
            $this->release(10);
        }
    }

#### 检查任务尝试次数

正如上面提到的，如果在处理过程中出现异常，任务会自动释放回到队列。你可以使用 `attempts` 方法检任务已经被查尝运行的次数：

    public function handle(Mailer $mailer)
    {
        if ($this->attempts() > 3) {
            //
        }
    }

<a name="pushing-jobs-onto-the-queue"></a>
## 推送任务到队列

默认的 Laravel 控制器位于 `app/Http/Controllers/Controller.php`，使用了一个 `DispatchesJob` trait。这个 trait 提供了几个方法允许你方便地把任务推送到队列，下面就是一个
`dispatch` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send a reminder e-mail to a given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendReminderEmail(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $this->dispatch(new SendReminderEmail($user));
        }
    }

当然，有时候你可能希望从你的应用程序的某个地方分发一个任务，而不仅仅是从一个路由或者一个控制器。在那个情况下，你可以在你的应用程序的任何一个类中包含 `DispatchesJobs` trait来进入其多个分发方法例如，下面是一个使用该 trait的一个类：

    <?php

    namespace App;

    use Illuminate\Foundation\Bus\DispatchesJobs;

    class ExampleClass
    {
        use DispatchesJobs;
    }

#### 指定任务的执行队列

你可以指定一个任务应该被发送到的队列。

通过将任务推送到不同的队列，你可以“分类”你的排队任务，你甚至可以为不同的队列考虑不同数量的工作线程。这里不能将任务推送到队列配置文件中设置的不同队列“连接”
，而仅是同一个连接的不同的队列。使用任务实例的 `onQueue` 方法制定推送队列。该 `onQueue` 方法由 Laravel 中的 `App\Jobs\Job` 基类提供：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send a reminder e-mail to a given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendReminderEmail(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $job = (new SendReminderEmail($user))->onQueue('emails');

            $this->dispatch($job);
        }
    }

<a name="delayed-jobs"></a>
### 延时任务

有时候，你可能希望延时执行一个排队的任务。例如，你可能希望将一个在客户签约15分钟之后向其发送提醒邮件的任务排队。你可以使用任务类的 `delay` 方法完成这个，

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send a reminder e-mail to a given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendReminderEmail(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $job = (new SendReminderEmail($user))->delay(60);

            $this->dispatch($job);
        }
    }

在这个例子中，我们指明任务需要在队列中延时60秒才可以被工作线程再次获取。

> **注意:** Amazon SQS 服务最大的延时时间是15分钟。

<a name="dispatching-jobs-from-requests"></a>
### 从请求中分发任务

将 HTTP 请求映射成任务是非常普遍的。所有，为了免去你手动映射请求，Laravel 提供了一些辅助方法让这个做起更加容易。让我们看一下`DispatchesJobs` trait 中的`dispatchFrom` 方法。默认情况下，这个 trait 囊括在 Laravel 的基类控制器中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class CommerceController extends Controller
    {
        /**
         * Process the given order.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function processOrder(Request $request, $id)
        {
            // Process the request...

            $this->dispatchFrom('App\Jobs\ProcessOrder', $request);
        }
    }

这个方法会坚持给定任务类的构造器，并且从 HTTP 请求（或者任何其他 `ArrayAccess` 类）中提取变量去填充类的构造参数。所以，如果我们的任务类的构造器接收变量 `productId` ， 类任务线会尝试从 HTTP 请求中抽取参数  `productId`。

你也可以传递一个数组作为 `dispatchFrom` 方法的第三个参数。这个数组会被用来填充任何从请求中获取不到的参数：

    $this->dispatchFrom('App\Jobs\ProcessOrder', $request, [
        'taxPercentage' => 20,
    ]);

<a name="running-the-queue-listener"></a>
## 运行队列监听器

#### 启动队列监听器

Laravel 包含了一个 Artisan 命令用来运行推送到队列中新的任务。你可以用 `queue:listen` 命令运行监听器：

    php artisan queue:listen

你也可以指定监听器使用的队列连接：

    php artisan queue:listen connection

需要注意，一旦这个任务被启动，它会一直运行直到被手动停止。你也可以使用一个监控进程，例如 [Supervisor](http://supervisord.org/)，来保证监听器不停地运行。

#### 队列优先级

You may pass a comma-delimited list of queue connections to the `listen` job to set queue priorities:

    php artisan queue:listen --queue=high,low

In this example, jobs on the `high` queue will always be processed before moving onto jobs from the `low` queue.

#### Specifying The Job Timeout Parameter

You may also set the length of time (in seconds) each job should be allowed to run:

    php artisan queue:listen --timeout=60

#### Specifying Queue Sleep Duration

In addition, you may specify the number of seconds to wait before polling for new jobs:

    php artisan queue:listen --sleep=5

Note that the queue only "sleeps" if no jobs are on the queue. If more jobs are available, the queue will continue to work them without sleeping.

<a name="supervisor-configuration"></a>
### Supervisor Configuration

Supervisor is a process monitor for the Linux operating system, and will automatically restart your `queue:listen` or `queue:work` commands if they fail. To install Supervisor on Ubuntu, you may use the following command:

    sudo apt-get install supervisor

Supervisor configuration files are typically stored in the `/etc/supervisor/conf.d` directory. Within this directory, you may create any number of configuration files that instruct supervisor how your processes should be monitored. For example, let's create a `laravel-worker.conf` file that starts and monitors a `queue:work` process:

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --daemon
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

In this example, the `numprocs` directive will instruct Supervisor to run 8 `queue:work` processes and monitor all of them, automatically restarting them if they fail. Once the configuration file has been created, you may update the Supervisor configuration and start the processes using the following commands:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

For more information on configuring and using Supervisor, consult the [Supervisor documentation](http://supervisord.org/index.html). Alternatively, you may use [Laravel Forge](https://forge.laravel.com) to automatically configure and manage your Supervisor configuration from a convenient web interface.

<a name="daemon-queue-listener"></a>
### Daemon Queue Listener

The `queue:work` Artisan command includes a `--daemon` option for forcing the queue worker to continue processing jobs without ever re-booting the framework. This results in a significant reduction of CPU usage when compared to the `queue:listen` command:

To start a queue worker in daemon mode, use the `--daemon` flag:

    php artisan queue:work connection --daemon

    php artisan queue:work connection --daemon --sleep=3

    php artisan queue:work connection --daemon --sleep=3 --tries=3

As you can see, the `queue:work` job supports most of the same options available to `queue:listen`. You may use the `php artisan help queue:work` job to view all of the available options.

#### Coding Considerations For Daemon Queue Listeners

Daemon queue workers do not restart the framework before processing each job. Therefore, you should be careful to free any heavy resources before your job finishes. For example, if you are doing image manipulation with the GD library, you should free the memory with `imagedestroy` when you are done.

Similarly, your database connection may disconnect when being used by long-running daemon. You may use the `DB::reconnect` method to ensure you have a fresh connection.

<a name="deploying-with-daemon-queue-listeners"></a>
### Deploying With Daemon Queue Listeners

Since daemon queue workers are long-lived processes, they will not pick up changes in your code without being restarted. So, the simplest way to deploy an application using daemon queue workers is to restart the workers during your deployment script. You may gracefully restart all of the workers by including the following command in your deployment script:

    php artisan queue:restart

This command will gracefully instruct all queue workers to restart after they finish processing their current job so that no existing jobs are lost.

> **Note:** This command relies on the cache system to schedule the restart. By default, APCu does not work for CLI jobs. If you are using APCu, add `apc.enable_cli=1` to your APCu configuration.

<a name="dealing-with-failed-jobs"></a>
## Dealing With Failed Jobs

Since things don't always go as planned, sometimes your queued jobs will fail. Don't worry, it happens to the best of us! Laravel includes a convenient way to specify the maximum number of times a job should be attempted. After a job has exceeded this amount of attempts, it will be inserted into a `failed_jobs` table. The name of the failed jobs can be configured via the `config/queue.php` configuration file.

To create a migration for the `failed_jobs` table, you may use the `queue:failed-table` command:

    php artisan queue:failed-table

When running your [queue listener](#running-the-queue-listener), you may specify the maximum number of times a job should be attempted using the `--tries` switch on the `queue:listen` command:

    php artisan queue:listen connection-name --tries=3

<a name="failed-job-events"></a>
### Failed Job Events

If you would like to register an event that will be called when a queued job fails, you may use the `Queue::failing` method. This event is a great opportunity to notify your team via e-mail or [HipChat](https://www.hipchat.com). For example, we may attach a callback to this event from the `AppServiceProvider` that is included with Laravel:

    <?php

    namespace App\Providers;

    use Queue;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function ($connection, $job, $data) {
                // Notify team of failing job...
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

#### Failed Method On Job Classes

For more granular control, you may define a `failed` method directly on a queue job class, allowing you to perform job specific actions when a failure occurs:

    <?php

    namespace App\Jobs;

    use App\Jobs\Job;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Bus\SelfHandling;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendReminderEmail extends Job implements SelfHandling, ShouldQueue
    {
        use InteractsWithQueue, SerializesModels;

        /**
         * Execute the job.
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function handle(Mailer $mailer)
        {
            //
        }

        /**
         * Handle a job failure.
         *
         * @return void
         */
        public function failed()
        {
            // Called when the job is failing...
        }
    }

<a name="retrying-failed-jobs"></a>
### Retrying Failed Jobs

To view all of your failed jobs that have been inserted into your `failed_jobs` database table, you may use the `queue:failed` Artisan command:

    php artisan queue:failed

The `queue:failed` command will list the job ID, connection, queue, and failure time. The job ID may be used to retry the failed job. For instance, to retry a failed job that has an ID of 5, the following command should be issued:

    php artisan queue:retry 5

If you would like to delete a failed job, you may use the `queue:forget` command:

    php artisan queue:forget 5

To delete all of your failed jobs, you may use the `queue:flush` command:

    php artisan queue:flush
