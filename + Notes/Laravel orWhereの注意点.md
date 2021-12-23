#php/laravel/tips 

同じコラムに対するorWhereのミス

```php
User::where('gender', 'Male')
	->where('birth_date', 'Male')
	->orWhere('birth_date', '<=', now()->subYears(18))
	->get();

// 間違ったquery(Male and under 65) or (over 18)

// (Male) and (under 65 or over 18)はこっち

User::where('gender', 'Male')
	->where(function($query) {
		->where(function($query)) {
		$query->where('birth_date', '>', now()->subYears(65))
			->orWhere('birth_date', '<=', now()->subYears(18));
			}
	})