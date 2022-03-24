---
marp: true
theme: yum
---

<!-- _class: title -->
# かしまさん用資料

---
<!-- class: content -->
# はじめに

- try-catch, DB transactionが書けてて凄い!
- 自己流さを感じるが、全体的にGood!
- 細かい所に目を配れるようになれば完ぺき！
- コードフォーマットなどエディタの便利な機能使えばもっと<br>楽になるかも！

---

<!-- _class: title -->
# 初級編

１個ずつコツコツ進めていこう！

---

# チュートリアル見ても良いかも？

- [Laravel+TDDの基礎](https://laravel-tdd.doc.tacck.net/)
  - Laravelの基本的なお作法的なもの
  - テストの書き方も学べる

---

# 便利なCollectionを使ってみよう！

- phpのarrayではなく、[Collection](https://readouble.com/laravel/8.x/ja/collections.html)でもっとスマートに書けるかも？

---

# 頑張ってテストを書いてみよう！

- テストを書くことで目視の確認をSKIPできる
- どこか修正した際に他の部分でエラーが吐いていないかが分かる
- 最初は時間がかかるが、最終的な開発時間は短くなる！
  - 変なバグがなくなる
- [LaravelビギナーのためのPHPUnitを利用したユニットテスト入門](https://reffect.co.jp/laravel/phpunit-test)
  - ここが一番やさしいかも

---

<!-- _class: title -->
# 中級編

余裕がでてきたら挑戦

---

# CodeSnifferの導入

- [コーディング規約PSR12への対応とphp_codesniffer](https://laravuejs.dev/tutorials/laravel-s1/laravel-115/)
- コードのスタイルで問題がある部分の提示・修正を行ってくれる
  - わざわざお手製でスペースを入れること無く勝手に修正してくれるツール

---

# MVC + Service層という構成にしてみよう

- [Service層を意識したLaravelのMVCモデル（概念編）](https://qiita.com/yukachin0414/items/3afafa63bd0796f8f3df)
  - Model, View, Controllerに加えてService層を採用する
  - Modelにロジックを書いても良いが、管理しづらくなってしまう
  - `insertGetResult`とかはServiceでもいいかも

---

# DBの設計を学ぼう！

- [マンガで分かるデータベース](https://www.amazon.co.jp/dp/4274066312)
  - マンガで基礎を入れた後に・・・
- [楽々ERDレッスン](https://www.amazon.co.jp/dp/4798110663)
  - ちょっと難しいけれども実践的な本を読むといいかも！

---

<!-- _class: title -->
# 上級編

ここまでくれば実務級！！

---

# PHPStan(LaraStan)の導入

- [LarastanでLaravelプロジェクトを静的解析しよう！](https://qiita.com/MasaKu/items/7ed6636a57fae12231e0)
- 静的解析ツール
  - 動かす前に問題がありそうなコードを教えてくれる
- コードレビューをお願いしないでもある程度コードの質が高くなる

---

# PHPの型を付ける

- [PHPで型を意識し安全なプログラムを書く](https://qiita.com/minato-naka/items/cc9da5fc1bc8cc4240a8)
  - 型を付けることで安全性の高いコードを書くことができる
