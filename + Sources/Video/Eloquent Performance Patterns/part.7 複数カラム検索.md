

```php

public funtion scopeSearch($query, string $terms = null)
{
	collect(explode(' ', $terms))->filter()->each(function ($term) use ($query)) {
		$term = '%'. $term. '%';
		$query->where(function ($query) use ($tern) {
			$query->where('first_name', 'like', $term)
				->orWhere('last_name', 'like', $term)
				->orWhereHas('company', function ($query) use ($term) {
					$query->where('name', 'like', $term');
				});
		});
	});
}


```