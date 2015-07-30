# 事件

- [介绍](#introduction)
- [注册事件 / 监听器](#registering-events-and-listeners)
- [定义事件](#defining-events)
- [定义监听器](#defining-listeners)
	- [队列事件监听器](#queued-event-listeners)
- [激发事件](#firing-events)
- [广播事件](#broadcasting-events)
	- [配置](#broadcast-configuration)
	- [标记为广播事件](#marking-events-for-broadcast)
	- [广播数据](#broadcast-data)
	- [消费事件广播](#consuming-event-broadcasts)
- [事件订阅](#event-subscribers)

<a name="introduction"></a>
## 介绍

Laravel 事件提供一个简单的观察者模式的实现，允许你订阅和监听应用程序中的事件，事件类一般存储在 `app/Events` 目录中，而相应的监听器存储在 `app/Listeners`中。

<a name="registering-events-and-listeners"></a>
## 注册事件 / 监听器

Laravel 中的 `EventServiceProvider` 提供一个方便的地方，用于注册所有的事件监听器，`listen` 属性包含一个所有事件（键）和相应监听器（值）的数组，所以，你可以根据应用程序的需要添加事件到数组中，例如，让我们来添加 `PodcastWasPurchased` 事件：

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\PodcastWasPurchased' => [
            'App\Listeners\EmailPurchaseConfirmation',
        ],
    ];

### 生成事件 / 监听器类

当然，手动为每个事件和监听器生成文件很麻烦，另一种方式，你可以只添加监听器和事件到 `EventServiceProvider` 然后使用 `event:generate` 命令，这个命令将生成 `EventServiceProvider` 中列出的所有事件和监听器，当然，已经存在的事件和进监听器不包含在内：

    php artisan event:generate

<a name="defining-events"></a>
## 定义事件

一个事件类只是一个数据容器，持有与事件相关的信息，例如，假设一下我们生成的 `PodcastWasPurchased` 事件接受一个[Eloquent ORM](/docs/{{version}}/eloquent) 对象：

    <?php

    namespace App\Events;

    use App\Podcast;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;

    class PodcastWasPurchased extends Event
    {
        use SerializesModels;

        public $podcast;

        /**
         * Create a new event instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }
    }

正如你所见，这个事件类没有特殊逻辑，只是所购买的 `Podcast` 对象的一个容器，如果事件对象被 PHP 的 `serialize` 函数序列化，事件类引入的 `SerializesModels` trait 将会优雅地将所有 Eloquent 模型序列化。

<a name="defining-listeners"></a>
## 定义监听器

接下来，让我们来看一下示例事件对应的监听器，事件监听器通过 `handle` 方法接收相应的事件实例，`event:generate` 命令将自动导入正确的事件类且在 `handle` 方法上类型提示（type-hint）相应的事件类。在 `handle` 方法中，你可以执行任何必要的逻辑来响应事件：

    <?php

    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class EmailPurchaseConfirmation
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Handle the event.
         *
         * @param  PodcastWasPurchased  $event
         * @return void
         */
        public function handle(PodcastWasPurchased $event)
        {
            // Access the podcast using $event->podcast...
        }
    }

你的事件监听器还可以在构造函数上类型提示任何需要的依赖，所有的事件监听器都将通过 Laravel [服务容器](/docs/{{version}}/container) 来实例化，所以依赖将被自动注入：

    use Illuminate\Contracts\Mail\Mailer;

    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

#### 阻止事件传播

有时候，你可能希望阻止事件传播到其他监听器，你可以通过使监听器的 `handle` 方法返回 `false` 来做到这一点。

<a name="queued-event-listeners"></a>
### 队列事件监听器

需要将事件监听器加入[队列](/docs/{{version}}/queues)吗? 这个再容易不过了，只需要将 `ShouldQueue` 接口加入监听器类。由 `event:generate` Artisan 命令生成的监听器类已经将这个接口导入到了当前的命名空间中，所以你可以直接使用：

    <?php

    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class EmailPurchaseConfirmation implements ShouldQueue
    {
        //
    }

就是这样！现在，当监听器因为一个事件而被调用时，这个监听器将会被使用自 Laravel[队列系统](/docs/{{version}}/queues)的事件转发器自动加入队列。如果当监听器被队列执行的过程当中没有报出异常，这个被队列的任务将自动害处理完后被删除。

#### 手动访问队列

如果你需要手动访问底层队列任务的 `delete` 和 `release` 方法，你可以这样做。`Illuminate\Queue\InteractsWithQueue` trait 是默认导入生成的监听器中的，使你可以访问这些方法：

    <?php

    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class EmailPurchaseConfirmation implements ShouldQueue
    {
        use InteractsWithQueue;

        public function handle(PodcastWasPurchased $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="firing-events"></a>
## 激发事件


激发一个事件，你可以使用 `Event` [facade](/docs/{{version}}/facades)，传递一个事件的实例给 `fire` 方法。`fire` 方法将这个事件转发给相关的注册事件：

    <?php

    namespace App\Http\Controllers;

    use Event;
    use App\Podcast;
    use App\Events\PodcastWasPurchased;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $userId
         * @param  int  $podcastId
         * @return Response
         */
        public function purchasePodcast($userId, $podcastId)
        {
            $podcast = Podcast::findOrFail($podcastId);

            // Purchase podcast logic...

            Event::fire(new PodcastWasPurchased($podcast));
        }
    }

可选的，你可以使用全局的 `event` 辅助函数激发事件：

    event(new PodcastWasPurchased($podcast));

<a name="broadcasting-events"></a>
## 广播事件

在许多现代的网络应用程序中，网络套接字（web sockets）被用于实现实时、在线更新的用户界面。当服务器上一些数据被更新时，一个消息通常会通过 websocket 连接发送到客户端进行处理。

为了帮助你构建这些类型的应用程序，Laravel 让使用网络套接字「广播」事件变得很容易。广播 Laravel 事件允许你在服务器端代码与客户端 JavaScript 框架之间共享相同的事件名。

<a name="broadcast-configuration"></a>
### 配置

所有的事件广播配置选项都存储在 `config/broadcasting.php` 配置文件中，Laravel 默认支持一些广播驱动:[Pusher](https://pusher.com), [Redis](/docs/{{version}}/redis) 和一个用于本地开发和调试的 `log` 驱动，其中每个驱动都包含一个配置示例文件。

#### 事件广播必备条件

事件广播需要以下依赖：

- Pusher: `pusher/pusher-php-server ~2.0`
- Redis: `predis/predis ~1.0`

#### 队列必备条件

在广播事件之前，你还需要配置和运行一个[队列监听器](/docs/{{version}}/queues)，所有事件广播都是通过队列任务来执行的，所以应用程序的响应时间不会被严重影响：

<a name="marking-events-for-broadcast"></a>
### 标记为广播事件

为了告知 Laravel 一个给定的事件需要广播，事件类需要实现 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 接口，`ShouldBroadcast` 接口只需要你实现一个方法：`broadcastOn`，`broadcastOn`返回一个「渠道」名数组，事件将在这些渠道上广播：

    <?php

    namespace App\Events;

    use App\User;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated extends Event implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should be broadcast on.
         *
         * @return array
         */
        public function broadcastOn()
        {
            return ['user.'.$this->user->id];
        }
    }

然后，你只需要按照正常操作[激发事件](#firing-events)，一旦这个事件被激发，一个[队列任务](/docs/{{version}}/queues) 将通过你指定的广播驱动，将此事件自动广播出去。

<a name="broadcast-data"></a>
### 广播数据

当一个事件被广播时，其所有的 `public` 属性将自动被序列化且作为事件负载（payload）被广播，从而使你通过 JavaScript 程序可以访问其所有的公共数据。所以，例如事件只有一个 `$user` 属性，此属性包含一个 Eloquent 模型，广播负载将如下：

    {
        "user": {
            "id": 1,
            "name": "Jonathan Banks"
            ...
        }
    }

然而，如果你希望对广播负载有更细粒度的控制，你可以在事件中添加 `broadcastWith` 方法，此方法应该返回数组数据，表示你所希望广播的事件：

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['user' => $this->user->id];
    }

<a name="consuming-event-broadcasts"></a>
### 消费事件广播

#### Pusher

你可以通过 Pusher 的 JavaScript SDK 使用[Pusher](https://pusher.com)驱动方便地消费事件广播。例如，让我们在前一个示例中消费一个 `App\Events\ServerCreated` 事件：

    this.pusher = new Pusher('pusher-key');

    this.pusherChannel = this.pusher.subscribe('user.' + USER_ID);

    this.pusherChannel.bind('App\\Events\\ServerCreated', function(message) {
        console.log(message.user);
    });

#### Redis

如果你使用 Redis 广播器，你需要编写你自己的 Redis 发布 / 订阅模型来接收消息和使用网络套接字技术将其广播。例如，你可以选择使用流行的[Socket.io](http://socket.io) 库，此库由 Node 编写的。

使用 `socket.io` 和 `ioredis`  Node 库，你能很快地编写一个事件广播器，用于发布所有 Laravel 广播的事件：

    var app = require('http').createServer(handler);
    var io = require('socket.io')(app);

    var Redis = require('ioredis');
    var redis = new Redis();

    app.listen(6001, function() {
        console.log('Server is running!');
    });

    function handler(req, res) {
        res.writeHead(200);
        res.end('');
    }

    io.on('connection', function(socket) {
        //
    });

    redis.psubscribe('*', function(err, count) {
        //
    });

    redis.on('pmessage', function(subscribed, channel, message) {
        message = JSON.parse(message);
        io.emit(channel + ':' + message.event, message.data);
    });

<a name="event-subscribers"></a>
## 事件订阅器

事件订阅器是一种可以在其中订阅多个事件的类，你可以在单个类中定义多个事件处理器。订阅器应该定义一个 `subscribe` 方法，传入一个事件转发器实例：

    <?php

    namespace App\Listeners;

    class UserEventListener
    {
        /**
         * Handle user login events.
         */
        public function onUserLogin($event) {}

        /**
         * Handle user logout events.
         */
        public function onUserLogout($event) {}

        /**
         * Register the listeners for the subscriber.
         *
         * @param  Illuminate\Events\Dispatcher  $events
         * @return array
         */
        public function subscribe($events)
        {
            $events->listen(
                'App\Events\UserLoggedIn',
                'App\Listeners\UserEventListener@onUserLogin'
            );

            $events->listen(
                'App\Events\UserLoggedOut',
                'App\Listeners\UserEventListener@onUserLogout'
            );
        }

    }

#### 注册事件订阅器

一旦订阅器被定义好后，可以将其注册到事件转发器中，你可以使用 `EventServiceProvider` 上的 `$subscribe` 属性注册订阅器，例如，让我们来添加 `UserEventListener`。

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventListener',
        ];
    }
