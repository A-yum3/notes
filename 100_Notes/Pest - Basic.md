#php/pest 
https://pestphp.com/docs/underlying-test-case

- `uses()`
	- トレイトを呼び出したファイルに適応できる
	- `in()`をチェーンさせると引数フォルダに再帰的に適応する
	- 複数使える
	- `uses(TestCase::class, RefreshDatabase::class)->in(__DIR__);`で現在フォルダに再帰的に適応する