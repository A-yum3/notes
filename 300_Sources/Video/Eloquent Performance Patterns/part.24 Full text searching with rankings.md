- ランキングを使った全文検索

```php

public function index()
{
	$posts = Post::query()
		->with('author')
		->when(request('search'), function($query, $search) {
			$query->whereRaw('match(title, body) against(? in boolean mode)', [$search]);
		}, function ($query) {
			$query->latest('published_at');
		})
		->paginate();
}

```