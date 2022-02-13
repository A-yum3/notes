- 2点間の地理の距離を取得する方法

```php

public function scopeSelectDistanceTo($query, array $coodinates)
{
	if(is_null($query->getQuery()->colums)) {
		$query->select('*');
	}
	
	$query->selectRaw('ST_Distance(
		ST_SRID(Point(longitude, latitude), 4326),
		ST_SRID(Point(?, ?), 4326)
	) as distance', $coordinates);
}

```