- 複数のクエリを実行する利点をUNIONを使って実現する
- Indexの効くDerived Tableを使用して最適化する
- 

```php

public function scopeSearch($query, string $terms = null)
{
	collect(str_getcsv($terms, ' ', '"'))->filter()->each(function ($term) use ($query) {
		$term = $term.'%';
		$query->whereIn('id', function ($query) use ($term) {
			$query->select('id')
				->from(function ($query) use ($term) {
					$query->select('id')
						->from('users')
						->where('first_name', 'like', $term)
						->orWhere('last_name', 'like', $term)
						->union(
							$query->newQuery()
								->select('users.id')
								->from('users')
								->join('companies', 'companies.id', '=', 'users.company_id')
								->where('companies.name', 'like', $term)
						);
				}, 'matches');
		});
	});
}

```