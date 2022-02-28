- Belongs To Version
- Join (Fast)

```php
public function index()
{
	$users = User::query()
		->select('users.*')
		->join('companies', 'comanies.id', '=', 'users.company_id')
		->orderBy('companies.name')
		->with('company')
		->paginate();
}
```



- SubQuery

```php
public function index()
{
	$users = User::query()
		->orderBy(Company::select('name')
			->whereColumn('id', 'users.company_id')
			->orderBy('name')
		  	->take(1)
		)
		->with('company')
		->paginate();
}
```