- Modelに`scopeVisibleTo($query, User $user)`で対応する

```php
public function scopeVisibleTo($query, User $user)
{
	if($user->is_owner) {
		return;
	}
	
	$query->where('xxx_id', $user->id);
}