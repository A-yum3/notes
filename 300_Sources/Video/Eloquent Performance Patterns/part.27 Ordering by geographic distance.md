- 距離でソートを行う

```php
public function scopeOrderByDistanceTo($query, array $coordinates, string $direction = 'asc')
{
	$direction = strtolower($direction) === 'asc' ? 'asc' : 'desc';
	
	$query->orderByRaw('ST_Distance(
		location,
		ST_SRID(Point(?, ?), 4326)
	) '.$direction, $coordinates);
}

```