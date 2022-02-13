- filteringする方法

```php

public function scopeWithinDistanceTo($query, array $coordinates, int $distance)
{
	$query->whereRaw('ST_Distance(
		ST_SRID(Point(longitude, latitude), 4326),
		ST_SRID(Point(?, ?), 4326)
	) <= ?', [...$coordinates, $distance])
}

```

- 更に高速化するには・・・
	- point型を格納する

