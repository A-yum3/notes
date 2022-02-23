## Laravel withの絞り込みはwith先だけ

Laravelでよく利用する機会のある`with`ですが、この`with`に対する理解が浅かった為に障害を起こしてしまった反省と戒めとして記事に残しておきます。


### 問題

例えば下記のようなクエリがあるとします

```php
$posts = Post::with(['user' => function ($query) {
			$query->where('role', '=', RoleType::ADMIN);
		 }])
		 ->get();
```

感覚的には `RoleType::ADMIN`のものだけのPostが取得されるように見えますが、実際には**postsは全て取得され、roleがAdmin以外のものは$posts->userがnull**で返却されます。

with先だけwhereしてしまい、結果的には全て取ってしまうことになります。

### 解決策

#### whereHasを使う

```php
$posts = Post::whereHas('users', function ($query) {
			$query->where('role', '=', RoleType::ADMIN);
		 })
		 ->with('user')
		 ->get();
```

デメリットとしてはwhereHas句はExists句を発行するため、参照するデータが大きいケースだとslow queryになってしまう可能性があるということです。

####  whereInを使う

```php
$posts = Post::whereIn('user_id', function ($query) {
			$query->select('id')
				->from('user_id')
				->where('role', '=', RoleType::ADMIN)
		 })
		 ->with('user')
		 ->get();
```
		

whereInでid側を絞り込み、サブクエリの方でwhereを行うことでインデックスも使えるようになります。
whereHasはslow queryになる可能性を含んでいますが、こちらはスピードの面では高速になります。

#### whereInでqueryを使う(やりすぎ)

```php
$posts = Post::whereIn('user_id', User::query()
					  ->where('role', '=', RoleType::ADMIN)
					  ->pluck('id')
					  )
		 ->with('user')
		 ->get();
```

以上のようなクエリを複数使うという過激なアプローチもありますが、ここまでは必要ないと思っています。
少し複雑度が上がるのでバランスを考えると上記のwhereIn程度でとどめておく方がちょうどよいのかと感じました。

### まとめ

- withでのwhereはwith先だけしかwhereしかしないので、nullが返る
- whereInを使ってサブクエリを発行する方がwhereHasより高速になる
- テストケースを漏れなく作り、考慮漏れがないか検証を疎かにしない

#### 謝辞

迷惑をかけてしまった関係者の方、本当に申し訳ないです。