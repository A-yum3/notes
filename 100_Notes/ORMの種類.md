[[ORMの種類-Mind]]

# ORMの種類

## ActiveRecord

### 概要

> An object that wraps a row in a database table or view, encapsulates the database access, and adds domain logic on that data.
> 
>![[Pasted image 20220625103149.png]]
>
> 引用: https://www.martinfowler.com/eaaCatalog/activeRecord.html

DeepL訳:

> データベースのテーブルやビューの行をラップし、データベースへのアクセスをカプセル化し、そのデータにドメインロジックを追加するオブジェクトです。

ドメインオブジェクトにデータアクセスロジックを配置するイメージ。

ロジックとデータアクセスが密結合になってしまうためテストがしにくくなる（DB接続が必要になる）

### メリット

- 単純なCRUDのWebアプリ開発には向いているし、使いやすい
- 納期がキツイタイプのWebアプリで納品して終わり、のケースだと適している場合が多い
	- 持続的なソフトウェア開発を行わない

### デメリット

- 手続き型言語に近づいてしまう
- 
- 

## DataMapper

### 概要

> A layer of Mappers (473) that moves data between objects and a database while keeping them independent of each other and the mapper itself.
> 
> ![[Pasted image 20220625103805.png]]
> 
> 引用: https://martinfowler.com/eaaCatalog/dataMapper.html

DeepL訳:

> オブジェクトとデータベースの間で、互いに独立した状態を保ちながらデータを移動させ、マッパー自体も独立させる。

DataMapperは、メモリ内（実行中プログラム）のオブジェクトをデータベースから分離するソフトウェアのレイヤー。

DataMapperを使用することで、メモリ内（実行中のプログラム）のオブジェクトはデータベースが存在することを知る必要はないので、データベーススキーマの知識も必要でなくなる。

## ちなみにRepositoryはなんなの？

### 概要

> Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.
> 
> ![[Pasted image 20220625105631.png]]
> 
> 引用: https://martinfowler.com/eaaCatalog/repository.html

DeepL訳:

> ドメインオブジェクトにアクセスするためのコレクションのようなインターフェイスを使用して、ドメインとデータマッピング層の間を仲介する。

Repositoryはドメインとデータマッピングレイヤーの間を仲介し、メモリ内（実行中プログラム内）のドメインオブジェクトコレクションのように機能する。

概念的には、リポジトリはDBに保存されたオブジェクトのセットと、それらに対して実行される操作をカプセル化し、オブジェクト指向のビューを提供する。

### つまりどういうこと？

> -   Entityを効率的に永続化するためのオブジェクト
> -   DataMapperは単一のEntityを永続化するためのオブジェクト
> -   Repositoryは複数のEntityを永続化するためのオブジェクト
>
> 引用: https://qiita.com/okeyaki/items/37eb4b66bd8ef62c1fe8

```
TaskRepository {
	taskofId(id)
	taskofname(name)
	save(task)
	remove(task)
}
```
引用: 実践ドメイン駆動設計

コレクションのように扱い、「集約」という単位でデータを読み書きする。

しかし複雑なクエリを頑張って実現させてしまうとメンテナンスがしにくく、パフォーマンス上の問題も引き起こすので、CQSを適用してクエリを切り出すほうが良い。

### メリット

- Repositoryを呼び出すビジネスロジック層が、データモデルに依存しなくなる

### デメリット

- 難しい

### CQRS(Command Query Responsibility Segregation): コマンドクエリ責務分離

####  概要

> 「情報の参照に使用するモデルと更新に使用するモデルに異なるものを使用する」というアーキテクチャです。
> 
> 引用: https://little-hands.hatenablog.com/entry/2019/12/02/cqrs


更新のときはDDDのEntityやValueObjectを渡して、RepositoryでDBに更新を行なう。

参照のときは特定のユースケースに特化した値の型を定義する。DTOのようなもの。

#### メリット

> -   複数集約にまたがるデータを取得する際のコードがシンプルになり、メンテナンス性が高まる
> -   クエリパフォーマンスが上がる、チューニングしやすくなる
> -   複数集約の条件(where)で絞り込んでのページングができるようになる
>
> 引用: https://little-hands.hatenablog.com/entry/2019/12/02/cqrs

#### デメリット

> -   オブジェクトの属性が参照されている場所が追いにくくなる
> -   元はGetterの参照を追えば確実だったが、別の手段を考える必要が生まれる
> -   アーキテクチャ自体が複雑になり、解説が必要になる
>
> 引用: https://little-hands.hatenablog.com/entry/2019/12/02/cqrs

中でもやはり複雑性が増すのが一番のデメリットのようです。

## LaravelのEloquentでのシーン

### 両方主張がある

#### EloquentでRepositoryを使わない方がいい派

> ActiveRecord パターンはデータベースと密結合になる代わりに実装コストを下げるのがメリットであり、Repository パターンとは逆の特徴を持ちます。
> そのため、**ActiveRecord と Repository を組み合わせると、ActiveRecord パターンのメリットは得られなくなります**。
> 
>  (省略)
>  
>  ActiveRecord 系の OR マッパーが付属するフレームワークとして有名な **Rails や Laravel は、ヘキサゴナルアーキテクチャ・オニオンアーキテクチャ・クリーンアーキテクチャといった、Repository パターンを活かせる設計と相性がよくない**とも考えています。
>
> 引用: https://www.kanzennirikaisita.com/posts/data-access-patterns

たしかになぁ〜というお気持ち。

#### EloquentでもRepositoryを使ったほうが良い派

> One option to help Active Record be more elegant is using the Repository Pattern, which is following the Dependency Inversion Principle (DIP) rule. It allows you to separate layers. In the future, it will be easier to replace Eloquent with another tool. 
>
> 引用: https://tsh.io/blog/active-record-vs-data-mapper-patterns-in-php/?utm_source=pocket_mylist

DeepL訳: 

> Active Recordをよりエレガントにするためのオプションとして、DIP（Dependency Inversion Principle）ルールに従ったRepositoryパターンを使用することが挙げられます。これによってレイヤーを分離することができる。将来的には、Eloquentを他のツールに置き換えることも容易になるだろう。

とりあえずレイヤー分離しておこうよ！って感じの主張ですね。

Eloquentを別ツールに置き換える用途やテスト容易性を確保する面では良い、というニュアンスのように感じました。

Eloquentが一切ロジックを持たずに、全てRepositoryやドメインモデルでロジックを持つのであればよいのか・・・？と思ったりもしました。

#### 個人的には

どちらの主張もメリデメがあって分かるので、なんとも言い難いですが、Eloquentの可能性を信じてRepositoryを使わなくても良いのかなぁと思います。

Eloquentの利点はやはりスピードだと感じているので、規模が大きくなるのが予見されるのであればDoctrineでRepositoryを使ってもよいのかなぁ・・・と。

しかしEloquentのQueryBuilderとかでRepositoryを作ってもそれはそれで良さそうだと感じました。難しいですね・・・。

## 参考記事

- [martinFowler.com | ActiveRecord](https://www.martinfowler.com/eaaCatalog/activeRecord.html)
- [martinFowler.com | DataMapper](https://martinfowler.com/eaaCatalog/dataMapper.html)
- [martinFowler.com | Repository](https://martinfowler.com/eaaCatalog/repository.html)
- [The Active Record and Data Mappers of ORM Pattern](https://medium.com/oceanize-geeks/the-active-record-and-data-mappers-of-orm-pattern-eefb8262b7bb)
- [Webアプリケーションの構成に関する予備知識](https://qiita.com/okeyaki/items/37eb4b66bd8ef62c1fe8)
- [データアクセスのパターンと、ActiveRecord や Eloquant による Repository の実装について](https://www.kanzennirikaisita.com/posts/data-access-patterns)
- [やはりお前たちのRepositoryは間違っている](https://qiita.com/mikesorae/items/ff8192fb9cf106262dbf)
- [CQRS実践入門 [ドメイン駆動設計]](https://little-hands.hatenablog.com/entry/2019/12/02/cqrs)
- [Active Record vs Data Mapper patterns in PHP – a practical comparison](https://tsh.io/blog/active-record-vs-data-mapper-patterns-in-php/?utm_source=pocket_mylist)
