- join, sub Queryが有効

```php
public function index()
{
	$books = Book::query()
		->orderByDesc(function ($query) {
			$query->select('borrowed_date')
				->from('checkouts')
				->whereColumn('book_id', 'books.id')
				->latest('borrowed_date')
				->take(1)
		})
		->withLastCheckout()
		->with('lastCheckout.user')
		->paginate();
}
```

joinする方式
- 注意としては大量のデータを扱う際には早くはない
- もしこの手法がパフォーマンスに問題を与えたら、キャッシュを有効利用すべき
```php
public function index()
{
	$books = Book::query()
		->orderBy(User::select('name')
			  ->join('checkouts', 'checkouts.user_id', '=', 'users.id')
			  ->whereColumn('checkouts.book_id', 'books.id')
			  ->latest('checkouts.borrowed_date')
			  ->take(1)
		 )
		->withLastCheckout()
		->with('lastCheckout.user')
}
```
