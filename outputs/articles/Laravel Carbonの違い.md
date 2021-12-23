
### Carbon\\Carbon

https://carbon.nesbot.com/docs/

おなじみの日時ライブラリ。組み込みのDateTimeクラスを継承している。
**mutable**なので、インスタンスを直接変更するため注意が必要。

`Carbon::now()->copy()->addDay()`のようにインスタンスをコピーしてから使わないとコードのあちこちで変更があった時に追うのが辛くなる。

### Illuminate\\Support\\Carbon

`Carbon\Carbon`を拡張した日時ライブラリ。実装を見ると分かる通り、Laravel8だともう`setTestNow()` しか実装がない。 (テストで日時を固定できるメソッド)
https://laravel.com/api/8.x/Illuminate/Support/Carbon.html

### Carbon\\CarbonImmutable

Carbonの **immutable**版。組み込みのDateTimeImmutableクラスを継承している。
こちらは新しいインスタンスが返されるため、`copy()`する必要がない。
インスタンスが作られた一番最初の状態を持つので、コードを順に追う必要がなくなる。

### DateTime

https://www.php.net/manual/ja/class.datetime.php

Carbonの元になっているPHP組み込みの日時ライブラリ。
何か特別な理由がない限りはLaravelではCarbonを使った方が楽に感じる。

### DateTimeImmutable

https://www.php.net/manual/ja/class.datetimeimmutable.php

CarbonImmutableの元になっているPHP組み込みの日時ライブラリ。
何か特別な理由がない限りはLaravelではCarbonImmutableを使った方が楽に感じる。

### まとめ

出来ることならばCarbonImmutableを採用したい。
また、DateTimeを使うことで柔軟なインターフェース設計にできることを@mpywさんに教えていただきました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://t.co/iNHK3FiuHW">https://t.co/iNHK3FiuHW</a><br><br>interface と class で必ずしも一致させる必要がない点も注意してください！自分はモジュラモノリスプロジェクトで以下のようにしました<br><br>interface<br>引数: DateTimeInterface, 返り値: DateTimeImmutable<br><br>class<br>引数: DateTimeInterface, 返り値: CarbonImmutable</p>&mdash; 素人 (@mpyw) <a href="https://twitter.com/mpyw/status/1473236915805880321?ref_src=twsrc%5Etfw">December 21, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>