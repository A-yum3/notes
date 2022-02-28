- whereHasはsubQueryを発行するため遅くなる
	- joinする？
		- 少しだけ早くなる
	- `orWhereIn(column, Closure)`でサブクエリを発行する
		- 50%ほど早くなる
	- 各クエリを分離することでインデックスを有効に使い、高速化する

```php

public function scopeSearch($query, string $terms = null)
{
	collect(str_getcsv($terms, ' ', '"'))->filter()->each(function ($term) use ($query) {
		$term = $term.'%';
		$query->where(function ($query) use ($term) {
			$query->where('first_name', 'like', $term)
				->orWhere('last_name', 'like', $term)
				->orWhereIn('company_id', function ($query) use ($term) {
					$query->select('id')
						->from('companies')
						->where('name', 'like', $term);
				});
		});
	});
}

```