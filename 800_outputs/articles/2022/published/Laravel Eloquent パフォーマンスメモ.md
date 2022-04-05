## EloquentでもパフォーマンスMemo

### Laravel-Debugbarは必須

https://github.com/barryvdh/laravel-debugbar

Laravel Debugbarはクエリに掛かった時間を表示してくれたり、発行されたSQLを簡単に確認できるとても性能調査に秀でたツールです。

### EagerLoading(with)の工夫

リレーションを含む時EagerLoadingのwithを使っている人が多いと思いますが、withのデータが大きくなる場合は逆にメモリを大量に使用してしまい、遅くなってしまう、という注意点があります。

withを使う場合はできる限り不必要なカラムは絞るとベスト

```php
User::query()
	->with('posts:title, published_at,')
```

### Eloquentを使用しないときはtoBase()

生データだけ欲しいからEloquent Modelは要らないというケースもあると思います。その場合はchunkなどを使うケースもありますが、場合によっては使えない時もあります。その時は`toBase()`を使うことで生データを取得し、メモリ消費を抑えられます。

countを走らせる場合などまさにそうですね。Eloquent Modelは必要ではなく、クエリだけで完結させたいです。

```php
User::toBase()
	->selectRaw("count(case when role = 'Premium' then 1 end) as premium")
```

参考: https://infyom.com/blog/tobase-function-in-laravel-eloquent

### あとからEagerLoadingしたい時のload()

モデルを先に取ってしまったけど後々の機能追加であのテーブルもLoadingしたい！みたいなユースケースは多々あると思います。
その時に最初に全部EagerLoadingするのではなく、必要になった時に `load()`でLazy Eager Loadingしてあげると後からも取れます。

```php
$user->load('posts')
```

また、`setRelation`を使うことで双方向のリレーションも動的に設定できるので上手く使っていきたいところです。

```php
$post->comments->each->setRelation('post', $post);
```

https://nextat.co.jp/staff/archives/159

### 脱whereHas

`whereHas`はExsitsを発行してしまうため、一歩間違えてしまうと逆にとっても遅いクエリを発行してしまう場合があります。
その場合の対策として、withでEagerLoadingしてからwhereを行ったり、joinを行うと少しだけ早くなるのですが、whereInでサブクエリを発行したり、Derived Tableを使うことでインデックスを上手く使い改善できる可能性があります。

orWhereInを使う
```php
$query->where(function ($query) use ($content) {
	$query
		->where('title', 'like', $content)
		->orWhereIn('comment_id', function ($query) use ($content) {
		$query->select('id')
			->from('comments')
			->where('content', 'like', $content);
	});
});
```

Derived Table + Unionを使う
```php

$query->whereIn('id', function ($query) use ($content) {
	$query->select('id')
		->from(function ($query) use ($content) {
			$query->select('id')
				->from('users')
				->union(
					$query->newQuery()
						->select('users.id')
						->from('users')
						->join('books', 'books.id', '=', 'books.book_id')
						->where('books.name', 'like', $content)
				);
		}, 'matches');
})
```

### ファジー検索を高速化する

- 大文字小文字数字以外の文字をすべて正規表現で削除する
- `whereRaw`を使い、正規表現式を使用する
	- SQLインジェクションには注意
- ファジー検索はインデックスが使用できない問題
	- 正規表現用にテーブルにVirtual Columnを追加する

```php
$table->string('name_normalized')->virtualAs("regexp_replace(name, [^A-Za-z0-9], '')")->index()
```

### 複合インデックスを使う

- ページネーションを使用するときはorderByクエリを入れて順番を保証する
- orderByでもインデックスは使用される
- インデックスを付けられた順序で適応されるため注意する

```php
// この場合
$table->index(['first', 'second']);

// この書き方だと複合インデックスは使われない
Model::query()
	->orderBy('second')
	->orderBy('first')

```

### リレーションを含むorderBy

####  Has-One, Belongs-ToのorderBy

- joinを使う (高速)
```php
User::query()
	->select('users.*')
	->join('companies', 'companies.user_id', '=', 'users.id')
	->orderBy('companies.name')
```

- サブクエリを使う

```php
User::query()
	->orderBy(Company::select('name')
			  ->whereColumn('user_id', 'users.id')
			  ->orderBy('name')
			  ->take(1)
			 )
```

#### Has-Many

Has-Manyは単純にjoinしてorderByしただけでは重複が発生する可能性がある。

```php
User::query()
	->orderByDesc(Login::select('created_at')
		->whereColumn('user_id', 'users.id')
		->latest()
		->take(1)
	 )
```
1つだけ取得するように工夫する必要がある。

## まとめ

- Eloquentでパフォーマンスを考えるとQueryBuilderを使う傾向になる
- QueryBuilderを使いすぎると可読性が落ちる可能性があるのでプロジェクト間で制定する必要がある
- インデックスを意識しつつクエリを書く必要があるのでSQLへの理解は必要不可欠
	- サブクエリへの理解や導出表も上手く使えるとなおさら良い
- `whereRaw`などでDBの関数を使ったほうがパフォーマンスは良くなる事が多いので極めたかったらやはりSQLへの深い理解は必須
- EloquentよりはSQLでどうにかするみたいな小技が多かった
- QueryBuilderを使う際にはSQLインジェクションにも気をつける必要がある
- ORMでなんとかしようとしない
