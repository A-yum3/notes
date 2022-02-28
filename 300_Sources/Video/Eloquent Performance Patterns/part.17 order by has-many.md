- has-many version
- join

Bad
```php
public function index()
{
	$users = User::query()
		->select('users.*')
		->join('logins', 'logins.user_id', '=', 'users.id')
		->orderByDesc('logins.created_at') // BAD Practice(重複レコードが出る)
		->withLastLogin()
		->paginate();
}
```

Better
```php
public function index()
{
	$users = User::query()
		->select('users.*')
		->join('logins', 'logins.user_id', '=', 'users.id')
		->groupBy('users.id')
		->orderByRaw('max(logins.created_at) desc')
		->withLastLogin()
		->paginate();
}
```

Good
```php
public function index()
{
	$users = User::query()
		->orderByDesc(Login::select('created_at')
			->whereColumn('user_id', 'users.id')
		 	->latest()
		 	->take(1)
		 )
		->withLastLogin()
		->paginate()
}
```