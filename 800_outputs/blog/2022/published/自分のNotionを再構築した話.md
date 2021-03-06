## 歴史

2020/5/24に初めての有料利用となり、意外と前から利用しているNotionですが、最近まで放置しぎみだったので一念発起し再構築しようと感じました。


自分はアーリーアダプターくらいのマインドを持っていて、好奇心旺盛に色々試すタイプの人間です。


自分のNotionの使い方ですが、勉強メモに使っていたり、ブログとして使っていたり、お買物リストに使っていたり、結構幅広く利用していた感じでした。


[nextjs-notion-starter-kit](https://github.com/transitive-bullshit/nextjs-notion-starter-kit)や[Super](https://super.so/?utm_source=notioneverything)を使ったり[Wraptas(旧Anotion)](https://wraptas.com/)を使ってブログを作るなどもしていましたが、Managementとしてちゃんと使用した経歴はなかったように感じます。

詳しくはこっちの記事で

%%ブログのリンク%%

## 方針と実施


前にNotionを使っていた時は何時間も考え、試行錯誤したページを作った挙げ句、1~3日ほどで管理をやめてしまうケースが多々ありました。マネジメントルールを厳しくしすぎたり、工程を複雑にしてしまうと結局使用されなくなり、腐ってしまうケースが多い為、今回は**やらないこと**から先に決めることにしました。


### やらないこと


#### 学習メモのStack


学習メモはNotionに溜めていくと管理が辛くなった記憶があるので今回はNotionでは一切扱わないことに決めました。（管理手法が悪いのかもしれませんが・・・）

以下が自分の感じた点です。
- 学習メモをStackしていく
	- タグ分類で疲れる
	- メモが大きくなりすぎるとメンテナンスがされなくなる
	- メモが大きくなりすぎると見通しが悪く、読まなくなる
	- Notionの情報へのアクセス性がちょっと悪い
		- できればO(1)で決まって欲しい（わがまま）
	- 見返さないメモは意味ないのでは？と感じてきてしまった
	- 細かい粒度でメモを作るとしてもページが境界があるので面倒
	- 階層が深くなりすぎると分からなくなりやすい
		- これはAll in One でデータベースに全部入れてフィルターとビューで調整すればなんとかなるかも
- vim操作ができない

なので、今回は学習メモは全て[Obsidian](https://obsidian.md/)に委任することにしました。

%%ブログの記事%%

#### 細かい管理

ざっくり言い過ぎですが、細かい管理は行わないことにしました。


- ReadingList
	- Obsidianで管理するため、Notionでは扱わない
- 収支管理
	- Money Forwardで管理し、Notionでは入力も手間なため扱わない
	- 雑費管理という大まかな区切りでの支出管理は行う（無駄遣いの可視化のため）
- ブログ記事管理
	- Obsidianで管理する（Markdownで保存するのでGit管理が楽）
- 欲しい物リスト
	- 欲しい物が多すぎる上に、AmazonのWish Listで事足りている感じがするので
- Linked Databaseを用いた複雑なデータ管理
	- 構築済みテンプレートを使うならまだしも、1から構築するのはコストが高いので見送り
	- 既に構築されているテンプレートを購入するとかなら全然OK
		- Notion アンバサダーのYuji Tsuburayaさんの[Note](https://note.com/35d/n/n0b7160b0d961)とか購入して構築するなどが一番コスパ良さそう
- 凝ったダッシュボード作成
	- ダッシュボードはサクッと通り過ぎる事が多いので「かわいい」「かっこいい」などは考えずに作ります
- 資産管理
	- 管理するほど資産がありません


例としてはこのような感じになります。逆にするものはなんだよ？と思う方もいらっしゃると思いますが、基本的に**自分に対してマイナスになることの可視化**ができるように管理していきます。一種の自戒ツールです。


### やること

#### マイナスになることの管理


自分にとってマイナス影響を与えるものの管理をします。

#### 雑費出費管理


コンビニや近くのパン屋での出費を記録していきます。いわゆる[ラテマネー](https://www.jaccs.co.jp/lesson/moneyplan/0006/)というもので、ちりも積も出費を管理します


管理方法としてはデータベースにテンプレートとして(100円, 300円, 500円, 1000円)と入力されたものを用意しておき、消費時に近い金額のものを選んでpageを作成します。近い金額がない場合は複数組み合わせて作成します。columnとしてplaceという消費場所を予め選択肢として作成してはありますが、最悪選択しないでも問題ないようにしています。


あくまでも目的は**ラテマネー発生による出費の概算値の確認**なので、そこから外れないように障壁を限りなく減らします。（こういうリッチな管理ツールだとリッチにしたくなってしまいますが、リッチにすればするほど入力コストが増え、使われなくなってしまいます）


後はビューとして週毎・月毎・年毎でグループ分けをしたビューを作成して終わりです。簡単ですね。

%%ページの画像%%


#### お買い物リスト


自分は業務スーパーのヘビーユーザーなのですが、毎度買うものをリストに作成するのは面倒なためNotionでお買い物リストを管理しています。


管理方法ですが、データベースを作成しカレンダービューも作ります。


次にテンプレートで内側に以前買ったことのあるもの・恒常的に買うもの・Todoリストを作ります。


%%画像%%


テンプレート内部のリストではよく行く店舗で商品が置かれている順番・買う量・買うチェックボックス・買ったチェックボックス（カゴに入れた）を作成しています。


もう1つビューも作成し、フィルターで買うチェックボックスにチェックが入っているものだけを抽出して実際にお店で使えるようにしています。


Todo Listはテンプレート内部のデータベースに登録されていない、突発的に必要になったもの用です。


#### SubScription管理


ありがちなやつですが、現代ではSubScriptionを何でもかんでもしてしまうので管理しています。ちなみに[テンプレート](https://extns.notion.site/Npedia-49fddc69667c4963964e54a012b36ec7)はここから引っ張ってきました。


#### WebClip管理


NotionでのWebClip管理は個人的にかなり使い心地が良く重宝しています。Chrome拡張の[Save To Notion](https://chrome.google.com/webstore/detail/save-to-notion/ldmmifpegigmeammaeckplhnjbbpccmm)が優秀で、ワンクリックで保存できるのが便利でよく使っています。


Clip用のデータベースもそんなに凝ったものではなく、inbox（最初は全部ここに入るがCategory分けすると消える）とallビューくらいしか作っていません。


All In Oneタイプのデーターベースにしておけば後々参照したい時もデーターベース参照とフィルターでタグ分類毎に出せるので個人的にはよく利用しています。

%%画像%%


#### タスク管理


タスク管理をしないとすぐに忘れてしまうため、Notionで扱うことにしました。ただ無秩序にデータがストックされていっても嫌ですし、キレイに流れていって欲しいということでテンプレート選びに苦労しました。


選んだテンプレートとしては[Ultimate Tasks and Projects Template for Notion](https://thomasjfrank.com/templates/task-and-project-notion-template/)です。


リンクを見てもらえれば分かると思いますが、かなりリッチで使い勝手も良いです。（しかも無料）


ところどころのcalloutは英語なのでDeepLに翻訳をかけてから入れ直してます。とっても便利で満足しています。


#### その他

後は大したことはしていないためSummaryだけ書いておきます。

- Memos
	- All In One型のメモデータベース。アイデア系も生活メモ系もここに入れて、カテゴリで管理
- Habit Tracker
	- 習慣トラッカーですが、ゴミ捨ての日確認だったり、何曜日にここお掃除する！程度の簡易的なものです
	- [Minimalist Habit Tracker Template for Notion](https://thomasjfrank.com/templates/habit-tracker-notion-template/)これ使っています
- Stack Knowledge
	- 過去に使っていて溜まっている学習メモです。一旦ここに置いてはありますが、そのうち消す予定です
- キャリア関連
	- 就活的なものはここにまとめてあります
	- ただResumeみたいのはMarkdownで管理したいのでGithubで別に管理しています。あくまでも業界研究や企業調査等のデータを置いてある感じです


#### 簡易的ダッシュボード


ダッシュボードはあまり凝らずにバランスが良くなるように作成しました。ウィジェットは[Indify](https://indify.co/)で作成したものを利用しています。


ダッシュボードには各ページへのリンクとTaskリストのDaily Taskだけデータベース参照して表示させています。


%%画像%%


### 余談


ところどころ統一させているcover画像ですが、全てMarpでサクッと作りました。

一応テンプレートとして置いておきます。（2/23のアップデートでダークモードのカラーが変わっていたので地味に探すのに苦労しました）

```css
/* @theme notion */

@import 'default';

section {
  font-family: "NotoSansJP-Regular", "NotoSans";
  background-color: #191919;
  width: 1500px;
  height: 600px;
}

section.title {
  text-align: center;
}

section.title h1 {
  color: #FFFFFF;
  font-family: "07にくまるフォント", "NotoSans";
  font-size: 90px;
  padding-top: 24px;
}
```

アイコンは全て[Notion Icons](https://www.notion.vip/icons/)を利用しています。シンプルで事足ります。

## まとめ

- 学習メモはObsidian
- Notionで管理しすぎない・リッチにしすぎない
- テンプレートを流用して構築コストを抑える