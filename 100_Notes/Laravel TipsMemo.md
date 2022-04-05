## Laravel Tips Memo

### Queryを使う時はcloneでImmutableに扱う
```php
$query = Product::query();

$today = request()->q_date ?? today();
if($today){
 $query->where('created_at', $today);
}


$active_products = $query->where('status', 1)->get();
$inactive_products = $query->where('status', 0)->get();
// $query->where('status', 1)->where('status', 0)->get()になる
```

```php
$active_products = (clone $query)->where('status', 1)->get();
$inactive_products = (clone $query)->where('status', 0)->get();
```

### order by descはlatest(), 逆はoldest()
```php
User::latest()->get();
User::oldest()->get();
// 指定したカラムの場合
User::latest('start_at')->first();
```


### EloquentのfindはwhereInも包括している

```php
Product::whereIn('id', $ids)->get();
// Better
Product::find($ids);
```

### Eloquentのfindでカラムを絞る

```php
User::find([1, 2, 3], ['first_name', 'email']);
```