- N+1 問題
- 実用は微妙だが、コンセプトは面白い
- 適切なリレーションシップだけロードするようにし、それ以外は例外を投げる

```php
<?php
	
namespace App;

use Illuminate\Database\Eloquent\Model as Eloquent;

class model extends Eloquent
{
	public function getRelationshipFromMethod($name)
	{
		$class = get_class($this);
		throw new Exception("Lazy-loading relationships is not allowed($class::$name).");
	}
}

```