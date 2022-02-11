- has-Oneのorder by は2つアプローチがある
- joinする(高速)

```php
public function index()
{
	$users = User::query()
		->select('users.*')
		->join('companies', 'companies.user_id', '=', 'users.id')
		->orderBy('companies.name')
		->with('company')
		->paginate();
}

```

- サブクエリを使う

```php
public function index()
{
	$users = User::query()
		->orderBy(Company::select('name')
			  ->whereColumn('user_id', 'users.id')
			  ->orderBy('name')
			  ->take(1)
		 )
		->with('company')
		->paginate();
}

```