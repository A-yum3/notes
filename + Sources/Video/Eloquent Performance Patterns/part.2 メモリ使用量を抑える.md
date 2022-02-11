#source/video/eloquent-perfomance-patterns 

### part.2 メモリ使用量を抑える

`Model::query()`が前提

`->select()`でカラムを絞る

`->with(['user'])`の中のカラムもClosureで絞れる

有効なのはインデックスページやエクスポートなどの多くのデータベースレコードとやり取りするページ。
メモリ消費量にも気を配り導入する。

```php
Post::query()
	->select('id', 'title', 'slug', 'published_at')
	->with(['author' => function ($query) {
		$query->select('id', 'name');
	}])
```

省略版

```php
Post::query()
	->with('author:id, name')
```