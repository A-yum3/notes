- Nullを含む結果を並び替えたい場合の例

nullを後ろに並び替えたい

```php
public function index()
{
	$users = User::query()
		->when(request('sort') === 'town', function ($query) {
			$query->orderByRaw('town is null')
				->orderBy('town', request('direction'));
		})
		->orderBy('name')
		->paginate();
}
```

postgres patterns

```php
public function index()
{
	$users = User::query()
		->when(request('sort') === 'town', function ($query) {
			$direction = strtolower(request('direction')) === 'asc' ? 'asc' : 'desc';
			$query->orderByRaw('town '.$direction.' nulls last');
		})
		->orderBy('name')
		->paginate();
}
```

macro化する

```php AppserviceProvider.php

public function boot()
{
	Builder::macro('orderByNullsLast', function ($column, $direction = 'asc') {
		$column = $this->getGrammar()->wrap($column);
		$direction = strtolower($direction) === 'asc' ? 'asc' : 'desc';
		return $this->orderByRaw("$column $direction nulls last");
	})
}

```