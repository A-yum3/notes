#source/video/eloquent-perfomance-patterns 

- `with`で`Eager Loading`を使う
	- withでloadするものが多すぎる場合はBad Solutions
- `last_login_id`を取ることで `first()`を取らないアプローチ


- サブクエリを発行する
- withCastsでサブクエリなどのカラムをキャスト出来る

```php
User::query()
	->addSelect(['last_login_at' => Login::select('created_at')
				 ->whereColumn('user_id', 'users.id')
				 ->latest()
				 ->take(1)
				])
	->withCasts(['last_login_at' => 'datetime'])
```

使用する事が多いクエリであれば`scope`でModelに書く

```php
public function scopeWithLastLoginAt($query)
{
	$query->addSelect(['last_login_at' => Login::select('created_at')
					   ->whereColumn('user_id', 'users.id')
					   ->latest()
					   ->take(1)
					  ])
			->withCast(['last_login_at' => datetime])
}
```

---

- `withCasts()`
- `scopreXXX()`
- `query()`
- `take(1)`