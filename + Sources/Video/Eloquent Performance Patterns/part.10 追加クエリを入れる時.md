
前回に引き続き、効果的な方法もあるが、過激なアプローチ

クエリを複数使う

```php

public function scopeSearch($query, string $terms = null)
{
	collect(str_getcsv($terms, ' ', '"'))->filter()->each(function ($term) use ($query) {
		$term = $term.'%';
		$query->where(function ($query) use ($term) {
			$query->where('first_name', 'like', $term)
				->orWhere('last_name', 'like', $term)
				->orWhereIn('company_id', Company::query()
							->where('name', 'like', $term)
							->pluck('id')
				);
		});
	});
}

```