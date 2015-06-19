# Collections

- [介绍](#introduction)
- [创建集合](#creating-collections)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 介绍

`Illuminate\Support\Collection` 提供流畅方便的工具来操作数组型数据，例如，看下面的代码，我们使用`collect`方法，将数组转化成集合 ，然后对集合中的每一个项执行 `strtoupper`，最后移除所有为空的项：

	$collection = collect(['taylor', 'abigail', null])->map(function ($name) {
		return strtoupper($name);
	})
	->reject(function ($name) {
		return empty($name);
	});


如你所见，`Collection` 类允许你将它的所有方法串连起来以一种流式操作对底动数组执行 mapping 和 reducing 操作。通常，每一个 `Collection` 方法返回一个完整的 `Collection` 实例。


<a name="creating-collections"></a>
## 创建集合

正如上面所提到的，`collect` 方法为给定的数组返回一个新的 `Illuminate\Support\Collection` 实例，所以创建一个集合就是这么简单：

	$collection = collect([1, 2, 3]);

默认情况下，[Eloquent](/docs/{{version}}/eloquent) 模型集合通常 `Collection` 实例的形式返回，然而，请随意在任何方便你的程序的地方使用 `Collection` 类。

<a name="available-methods"></a>
## 可用方法

对于该文档剩余部分，我们将讨论 `Collection` 类上的每一个可用的方法。请记住，所有这些方法都可以串联起来流式地操作底层数组，而且每个一个方法都返回一个新的 `Collection` 实例，允许保存你在必要时保存一份集合的原始拷贝。

你可以从表格中选择任何方法来查看其使用方法的示例：

<style>
	#collection-method-list > p {
		column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
		column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
	}

	#collection-method-list a {
		display: block;
	}
</style>

<div id="collection-method-list" markdown="1">
[all](#method-all)
[chunk](#method-chunk)
[collapse](#method-collapse)
[contains](#method-contains)
[count](#method-count)
[diff](#method-diff)
[each](#method-each)
[filter](#method-filter)
[first](#method-first)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[isEmpty](#method-isempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[map](#method-map)
[merge](#method-merge)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[sum](#method-sum)
[take](#method-take)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[unique](#method-unique)
[values](#method-values)
[where](#method-where)
[whereLoose](#method-whereloose)
[zip](#method-zip)
</div>

<a name="method-listing"></a>
## 方法列表

<style>
	#collection-method code {
		font-size: 14px;
	}

	#collection-method:not(.first-collection-method) {
		margin-top: 50px;
	}
</style>

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

`all` 方法仅返回用集合表示的层底数组：

	collect([1, 2, 3])->all();

	// [1, 2, 3]

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

`chunk` 方法根据给定的尺寸将集合拆分为多个更小的集合：

	$collection = collect([1, 2, 3, 4, 5, 6, 7]);

	$chunks = $collection->chunk(4);

	$chunks->toArray();

	// [[1, 2, 3, 4], [5, 6, 7]]

当在[视图](/docs/{{version}}/views)使用[Bootstrap](http://getbootstrap.com/css/#grid)这样的网格系统时，这个方法尤其有用，想像一下你需要将一个[Eloquent](/docs/{{version}}/eloquent)模型集合显示到网格中：

	@foreach ($products->chunk(3) as $chunk)
		<div class="row">
			@foreach ($chunk as $product)
				<div class="col-xs-4">{{ $product->name }}</div>
			@endforeach
		</div>
	@endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

`collapse` 方法将多个数据合并为一个扁平的集合：

	$collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

	$collapsed = $collection->collapse();

	$collapsed->all();

	// [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-contains"></a>
#### `contains()` {#collection-method}

`contains` 方法用于判断集合是否包含某项：

	$collection = collect(['name' => 'Desk', 'price' => 100]);

	$collection->contains('Desk');

	// true

	$collection->contains('New York');

	// false

你也可以向 `contains` 方法中传入一个键值对，些方法用于判断给定键值对是否存在于集合中：

	$collection = collect([
		['product' => 'Desk', 'price' => 200],
		['product' => 'Chair', 'price' => 100],
	]);

	$collection->contains('product', 'Bookcase');

	// false

最后，你也可以向 `contains` 方法中传入一个回调函数来执行你自己的真值测试：

	$collection = collect([1, 2, 3, 4, 5]);

	$collection->contains(function ($key, $value) {
		return $value > 5;
	});

	// false

<a name="method-count"></a>
#### `count()` {#collection-method}

`count` 方法返回集合中项的总数：

	$collection = collect([1, 2, 3, 4]);

	$collection->count();

	// 4

<a name="method-diff"></a>
#### `diff()` {#collection-method}

`diff` 方法用于比较一个集合跟另外一个集合，或者一个 PHP `array`:

	$collection = collect([1, 2, 3, 4, 5]);

	$diff = $collection->diff([2, 4, 6, 8]);

	$diff->all();

	// [1, 3, 5]

<a name="method-each"></a>
#### `each()` {#collection-method}

`each` 方法遍历集合中每一项且将每一项传入给定的回调中：

	$collection = $collection->each(function ($item, $key) {
		//
	});

从回调中返回 `false` 退出循环：

	$collection = $collection->each(function ($item, $key) {
		if (/* some condition */) {
			return false;
		}
	});

<a name="method-filter"></a>
#### `filter()` {#collection-method}

`filter` 方法通过给定的回调函数来过滤集合，只保留通过真值过滤的项：

	$collection = collect([1, 2, 3, 4]);

	$filtered = $collection->filter(function ($item) {
		return $item > 2;
	});

	$filtered->all();

	// [3, 4]

`filter` 相对的，请查看[reject](collections#method-reject)方法：

<a name="method-first"></a>
#### `first()` {#collection-method}

`first` 方法返回集合中的第一个通过真值测试的项:

	collect([1, 2, 3, 4])->first(function ($key, $value) {
		return $value > 2;
	});

	// 3

你还可以调用无参的 `first` 方法获取集合中的第一项，如果集合为空，返回 `null`:

	collect([1, 2, 3, 4])->first();

	// 1

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

`flatten` 方法用于将多维集合转化为单维集合：

	$collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

	$flattened = $collection->flatten();

	$flattened->all();

	// ['taylor', 'php', 'javascript'];

<a name="method-flip"></a>
#### `flip()` {#collection-method}

`flip` 方法用于交换集合中的键与其相应的值：

	$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

	$flipped = $collection->flip();

	$flipped->all();

	// ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

`forget` 方法根据键从集合中移除项：

	$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

	$collection->forget('name');

	$collection->all();

	// [framework' => 'laravel']

> **注意:** 不像其它大多数集合方法，`forget` 方法不返回新集合，只修改被调用集合

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

`forPage` 方法根据页码返回一个包含所有应该显示的项的新集合：

	$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9])->forPage(2, 3);

	$collection->all();

	// [4, 5, 6]

此方法分别需要页码和每页需要显示的项数作为参数。

<a name="method-get"></a>
#### `get()` {#collection-method}

`get` 方法根据给定的 key 获取对应项，如果 key 不存在，返回 `null`:

	$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

	$value = $collection->get('name');

	// taylor

你可以选择是否传入一个默认值作为第二个参数：

	$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

	$value = $collection->get('foo', 'default-value');

	// default-value

你甚至可以传入一个回调作为默认值，如果指定的 key 不存在，则返回此回调的结果：

	$collection->get('email', function () {
		return 'default-value';
	});

	// default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

`groupBy` 方法根据给定的 key 对集合分组：

	$collection = collect([
		['account_id' => 'account-x10', 'product' => 'Chair'],
		['account_id' => 'account-x10', 'product' => 'Bookcase'],
		['account_id' => 'account-x11', 'product' => 'Desk'],
	]);

	$grouped = $collection->groupBy('account_id');

	$grouped->toArray();

	/*
		[
			'account-x10' => [
				['account_id' => 'account-x10', 'product' => 'Chair'],
				['account_id' => 'account-x10', 'product' => 'Bookcase'],
			],
			'account-x11' => [
				['account_id' => 'account-x11', 'product' => 'Desk'],
			],
		]
	*/

除了传入字符串 `key`，你还可以传入一个回调，改变分组的 key：

	$grouped = $collection->groupBy(function ($item, $key) {
		return substr($item['account_id'], -3);
	});

	$grouped->toArray();

	/*
		[
			'x10' => [
				['account_id' => 'account-x10', 'product' => 'Chair'],
				['account_id' => 'account-x10', 'product' => 'Bookcase'],
			],
			'x11' => [
				['account_id' => 'account-x11', 'product' => 'Desk'],
			],
		]
	*/

<a name="method-has"></a>
#### `has()` {#collection-method}

The `has` method determines if a given key exists in the collection:

	$collection = collect(['account_id' => 1, 'product' => 'Desk']);

	$collection->has('email');

	// false

<a name="method-implode"></a>
#### `implode()` {#collection-method}

The `implode` method joins the items in a collection. Its arguments depend on the type of items in the collection.

If the collection contains arrays or objects, you should pass the key of the attributes you wish to join, and the "glue" string you wish to place between the values:

	$collection = collect([
		['account_id' => 1, 'product' => 'Desk'],
		['account_id' => 2, 'product' => 'Chair'],
	]);

	$collection->implode('product', ', ');

	// Desk, Chair

If the collection contains simple strings or numeric values, simply pass the "glue" as the only argument to the method:

	collect([1, 2, 3, 4, 5])->implode('-');

	// '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

The `intersect` method removes any values that are not present in the given `array` or collection:

	$collection = collect(['Desk', 'Sofa', 'Chair']);

	$intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

	$intersect->all();

	// [0 => 'Desk', 2 => 'Chair']

As you can see, the resulting collection will preserve the original collection's keys.

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

The `isEmpty` method returns `true` if the collection is empty; otherwise, `false` is returned:

	collect([])->isEmpty();

	// true

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

Keys the collection by the given key:

	$collection = collect([
		['product_id' => 'prod-100', 'name' => 'desk'],
		['product_id' => 'prod-200', 'name' => 'chair'],
	]);

	$keyed = $collection->keyBy('product_id');

	$keyed->all();

	/*
		[
			'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
			'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
		]
	*/

If multiple items have the same key, only the last one will appear in the new collection.

You may also pass your own callback, which should return the value to key the collection by:

	$keyed = $collection->keyBy(function ($item) {
		return strtoupper($item['product_id']);
	});

	$keyed->all();

	/*
		[
			'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
			'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
		]
	*/


<a name="method-keys"></a>
#### `keys()` {#collection-method}

The `keys` method returns all of the collection's keys:

	$collection = collect([
		'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
		'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
	]);

	$keys = $collection->keys();

	$keys->all();

	// ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

The `last` method returns the last element in the collection that passes a given truth test:

	collect([1, 2, 3, 4])->last(function ($key, $value) {
		return $value < 3;
	});

	// 2

You may also call the `last` method with no arguments to get the last element in the collection. If the collection is empty, `null` is returned:

	collect([1, 2, 3, 4])->last();

	// 4

<a name="method-map"></a>
#### `map()` {#collection-method}

The `map` method iterates through the collection and passes each value to the given callback. The callback is free to modify the item and return it, thus forming a new collection of modified items:

	$collection = collect([1, 2, 3, 4, 5]);

	$multiplied = $collection->map(function ($item, $key) {
		return $item * 2;
	});

	$multiplied->all();

	// [2, 4, 6, 8, 10]

> **Note:** Like most other collection methods, `map` returns a new collection instance; it does not modify the collection it is called on. If you want to transform the original collection, use the [`transform`](#method-transform) method.

<a name="method-merge"></a>
#### `merge()` {#collection-method}

The `merge` method merges the given array into the collection. Any string key in the array matching a string key in the collection will overwrite the value in the collection:

	$collection = collect(['product_id' => 1, 'name' => 'Desk']);

	$merged = $collection->merge(['price' => 100, 'discount' => false]);

	$merged->all();

	// ['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]

If the given array's keys are numeric, the values will be appended to the end of the collection:

	$collection = collect(['Desk', 'Chair']);

	$merged = $collection->merge(['Bookcase', 'Door']);

	$merged->all();

	// ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

The `pluck` method retrieves all of the collection values for a given key:

	$collection = collect([
		['product_id' => 'prod-100', 'name' => 'Desk'],
		['product_id' => 'prod-200', 'name' => 'Chair'],
	]);

	$plucked = $collection->pluck('name');

	$plucked->all();

	// ['Desk', 'Chair']

You may also specify how you wish the resulting collection to be keyed:

	$plucked = $collection->pluck('name', 'product_id');

	$plucked->all();

	// ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

The `pop` method removes and returns the last item from the collection:

	$collection = collect([1, 2, 3, 4, 5]);

	$collection->pop();

	// 5

	$collection->all();

	// [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

The `prepend` method adds an item to the beginning of the collection:

	$collection = collect([1, 2, 3, 4, 5]);

	$collection->prepend(0);

	$collection->all();

	// [0, 1, 2, 3, 4, 5]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

The `pull` method removes and returns an item from the collection by its key:

	$collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

	$collection->pull('name');

	// 'Desk'

	$collection->all();

	// ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

The `push` method appends an item to the end of the collection:

	$collection = collect([1, 2, 3, 4]);

	$collection->push(5);

	$collection->all();

	// [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

The `put` method sets the given key and value in the collection:

	$collection = collect(['product_id' => 1, 'name' => 'Desk']);

	$collection->put('price', 100);

	$collection->all();

	// ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

The `random` method returns a random item from the collection:

	$collection = collect([1, 2, 3, 4, 5]);

	$collection->random();

	// 4 - (retrieved randomly)

You may optionally pass an integer to `random`. If that integer is more than `1`, a collection of items is returned:

	$random = $collection->random(3);

	$random->all();

	// [2, 4, 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

The `reduce` method reduces the collection to a single value, passing the result of each iteration into the subsequent iteration:

	$collection = collect([1, 2, 3]);

	$total = $collection->reduce(function ($carry, $item) {
		return $carry + $item;
	});

	// 6

The value for `$carry` on the first iteration is `null`; however, you may specify its initial value by passing a second argument to `reduce`:

	$collection->reduce(function ($carry, $item) {
		return $carry + $item;
	}, 4);

	// 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

The `reject` method filters the collection using the given callback. The callback should return `true` for any items it wishes to remove from the resulting collection:

	$collection = collect([1, 2, 3, 4]);

	$filtered = $collection->reject(function ($item) {
		return $item > 2;
	});

	$filtered->all();

	// [1, 2]

For the inverse of the `reject` method, see the [`filter`](#method-filter) method.

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

The `reverse` method reverses the order of the collection's items:

	$collection = collect([1, 2, 3, 4, 5]);

	$reversed = $collection->reverse();

	$reversed->all();

	// [5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()` {#collection-method}

The `search` method searches the collection for the given value and returns its key if found. If the item is not found, `false` is returned.

	$collection = collect([2, 4, 6, 8]);

	$collection->search(4);

	// 1

The search is done using a "loose" comparison. To use strict comparison, pass `true` as the second argument to the method:

	$collection->search('4', true);

	// false

Alternatively, you may pass in your own callback to search for the first item that passes your truth test:

	$collection->search(function ($item, $key) {
		return $item > 5;
	});

	// 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

The `shift` method removes and returns the first item from the collection:

	$collection = collect([1, 2, 3, 4, 5]);

	$collection->shift();

	// 1

	$collection->all();

	// [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

The `shuffle` method randomly shuffles the items in the collection:

	$collection = collect([1, 2, 3, 4, 5]);

	$shuffled = $collection->shuffle();

	$shuffled->all();

	// [3, 2, 5, 1, 4] // (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

The `slice` method returns a slice of the collection starting at the given index:

	$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

	$slice = $collection->slice(4);

	$slice->all();

	// [5, 6, 7, 8, 9, 10]

If you would like to limit the size of the returned slice, pass the desired size as the second argument to the method:

	$slice = $collection->slice(4, 2);

	$slice->all();

	// [5, 6]

The returned slice will have new, numerically indexed keys. If you wish to preserve the original keys, pass `true` as the third argument to the method.

<a name="method-sort"></a>
#### `sort()` {#collection-method}

The `sort` method sorts the collection:

	$collection = collect([5, 3, 1, 2, 4]);

	$sorted = $collection->sort();

	$sorted->values()->all();

	// [1, 2, 3, 4, 5]

The sorted collection keeps the original array keys. In this example we used the [`values`](#method-values) method to reset the keys to consecutively numbered indexes.

For sorting a collection of nested arrays or objects, see the [`sortBy`](#method-sortby) and [`sortByDesc`](#method-sortbydesc) methods.

If your sorting needs are more advanced, you may pass a callback to `sort` with your own algorithm. Refer to the PHP documentation on [`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters), which is what the collection's `sort` method calls under the hood.

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

The `sortBy` method sorts the collection by the given key:

	$collection = collect([
		['name' => 'Desk', 'price' => 200],
		['name' => 'Chair', 'price' => 100],
		['name' => 'Bookcase', 'price' => 150],
	]);

	$sorted = $collection->sortBy('price');

	$sorted->values()->all();

	/*
		[
			['name' => 'Chair', 'price' => 100],
			['name' => 'Bookcase', 'price' => 150],
			['name' => 'Desk', 'price' => 200],
		]
	*/

The sorted collection keeps the original array keys. In this example we used the [`values`](#method-values) method to reset the keys to consecutively numbered indexes.

You can also pass your own callback to determine how to sort the collection values:

	$collection = collect([
		['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
		['name' => 'Chair', 'colors' => ['Black']],
		['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
	]);

	$sorted = $collection->sortBy(function ($product, $key) {
		return count($product['colors']);
	});

	$sorted->values()->all();

	/*
		[
			['name' => 'Chair', 'colors' => ['Black']],
			['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
			['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
		]
	*/

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

This method has the same signature as the [`sortBy`](#method-sortby) method, but will sort the collection in the opposite order.

<a name="method-splice"></a>
#### `splice()` {#collection-method}

The `splice` method removes and returns a slice of items starting at the specified index:

	$collection = collect([1, 2, 3, 4, 5]);

	$chunk = $collection->splice(2);

	$chunk->all();

	// [3, 4, 5]

	$collection->all();

	// [1, 2]

You may pass a second argument to limit the size of the resulting chunk:

	$collection = collect([1, 2, 3, 4, 5]);

	$chunk = $collection->splice(2, 1);

	$chunk->all();

	// [3]

	$collection->all();

	// [1, 2, 4, 5]

In addition, you can pass a third argument containing the new items to replace the items removed from the collection:

	$collection = collect([1, 2, 3, 4, 5]);

	$chunk = $collection->splice(2, 1, [10, 11]);

	$chunk->all();

	// [3]

	$collection->all();

	// [1, 2, 10, 11, 4, 5]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

The `sum` method returns the sum of all items in the collection:

	collect([1, 2, 3, 4, 5])->sum();

	// 15

If the collection contains nested arrays or objects, you should pass a key to use for determining which values to sum:

	$collection = collect([
		['name' => 'JavaScript: The Good Parts', 'pages' => 176],
		['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
	]);

	$collection->sum('pages');

	// 1272

In addition, you may pass your own callback to determine which values of the collection to sum:

	$collection = collect([
		['name' => 'Chair', 'colors' => ['Black']],
		['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
		['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
	]);

	$collection->sum(function ($product) {
		return count($product['colors']);
	});

	// 6

<a name="method-take"></a>
#### `take()` {#collection-method}

The `take` method returns a new collection with the specified number of items:

	$collection = collect([0, 1, 2, 3, 4, 5]);

	$chunk = $collection->take(3);

	$chunk->all();

	// [0, 1, 2]

You may also pass a negative integer to take the specified amount of items from the end of the collection:

	$collection = collect([0, 1, 2, 3, 4, 5]);

	$chunk = $collection->take(-2);

	$chunk->all();

	// [4, 5]

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

The `toArray` method converts the collection into a plain PHP `array`. If the collection's values are [Eloquent](/docs/{{version}}/eloquent) models, the models will also be converted to arrays:

	$collection = collect(['name' => 'Desk', 'price' => 200]);

	$collection->toArray();

	/*
		[
			['name' => 'Desk', 'price' => 200],
		]
	*/

> **Note:** `toArray` also converts all of its nested objects to an array. If you want to get the underlying array as is, use the [`all`](#method-all) method instead.

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

The `toJson` method converts the collection into JSON:

	$collection = collect(['name' => 'Desk', 'price' => 200]);

	$collection->toJson();

	// '{"name":"Desk","price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

The `transform` method iterates over the collection and calls the given callback with each item in the collection. The items in the collection will be replaced by the values returned by the callback:

	$collection = collect([1, 2, 3, 4, 5]);

	$collection->transform(function ($item, $key) {
		return $item * 2;
	});

	$collection->all();

	// [2, 4, 6, 8, 10]

> **Note:** Unlike most other collection methods, `transform` modifies the collection itself. If you wish to create a new collection instead, use the [`map`](#method-map) method.

<a name="method-unique"></a>
#### `unique()` {#collection-method}

The `unique` method returns all of the unique items in the collection:

	$collection = collect([1, 1, 2, 2, 3, 4, 2]);

	$unique = $collection->unique();

	$unique->values()->all();

	// [1, 2, 3, 4]

The returned collection keeps the original array keys. In this example we used the [`values`](#method-values) method to reset the keys to consecutively numbered indexes.

When dealing with nested arrays or objects, you may specify the key used to determine uniqueness:

	$collection = collect([
		['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
		['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
		['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
		['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
		['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
	]);

	$unique = $collection->unique('brand');

	$unique->values()->all();

	/*
		[
			['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
			['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
		]
	*/

You may also pass your own callback to determine item uniqueness:

	$unique = $collection->unique(function ($item) {
		return $item['brand'].$item['type'];
	});

	$unique->values()->all();

	/*
		[
			['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
			['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
			['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
			['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
		]
	*/

<a name="method-values"></a>
#### `values()` {#collection-method}

The `values` method returns a new collection with the keys reset to consecutive integers:

	$collection = collect([
		10 => ['product' => 'Desk', 'price' => 200],
		11 => ['product' => 'Desk', 'price' => 200]
	]);

	$values = $collection->values();

	$values->all();

	/*
		[
			0 => ['product' => 'Desk', 'price' => 200],
			1 => ['product' => 'Desk', 'price' => 200],
		]
	*/
<a name="method-where"></a>
#### `where()` {#collection-method}

The `where` method filters the collection by a given key / value pair:

	$collection = collect([
		['product' => 'Desk', 'price' => 200],
		['product' => 'Chair', 'price' => 100],
		['product' => 'Bookcase', 'price' => 150],
		['product' => 'Door', 'price' => 100],
	]);

	$filtered = $collection->where('price', 100);

	$filtered->all();

	/*
	[
		['product' => 'Chair', 'price' => 100],
		['product' => 'Door', 'price' => 100],
	]
	*/

The `where` method uses strict comparisons when checking item values. Use the [`whereLoose`](#where-loose) method to filter using "loose" comparisons.

<a name="method-whereloose"></a>
#### `whereLoose()` {#collection-method}

This method has the same signature as the [`where`](#method-where) method; however, all values are compared using "loose" comparisons.

<a name="method-zip"></a>
#### `zip()` {#collection-method}

The `zip` method merges together the values of the given array with the values of the collection at the corresponding index:

	$collection = collect(['Chair', 'Desk']);

	$zipped = $collection->zip([100, 200]);

	$zipped->all();

	// [['Chair', 100], ['Desk', 200]]
