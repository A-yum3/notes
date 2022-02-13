- DDD
- ヘキサゴナルアーキテクチャ
- イベント・ソーシング

> 人間はカテゴリー別に考えるもの。コードもそれを反映したものであるべき

### ドメイン

- ドメイン
	- 解決しようとしているビジネス上の問題の集合
	- プロジェクト内のコンセプトをグループ化することを目的


- どこまでやるか問題
- コントローラとモデルの間に1対1のマッピングが存在しない

- 以下で構成される
	- モデル
	- クエリビルダ
	- ドメインイベント
	- 検証ルール


```example
One specific domain folder per business concept
src/Domain/Invoices/
    ├── Actions 
 	├── QueryBuilders
    ├── Collections
    ├── DataTransferObjects
    ├── Events
    ├── Exceptions
    ├── Listeners
    ├── Models
    ├── Rules
    └── States
 

src/Domain/Customers/
	//...
```

```example 
The admin HTTP application 

src/App/Admin/
    ├── Controllers 
 	├── Middlewares
    ├── Requests
    ├── Resources
    └── ViewModels 

The REST API application 

src/App/Api/
    ├── Controllers 
 	├── Middlewares
    ├── Requests
    └── Resources 

The console application 

src/App/Console/
    └── Commands
```

構造は本題でないので、好きなように構築することができる

もしルートの名前空間を分けたい場合は `composer.json`を変更することでできる

```json
{
	"autoload" : {
		"psr-4" : {
			"App\\" "src/App/",
			"Domain\\" : "src/Domain/",
			"Support\\" : "src/Support/" // ヘルパーの置き場所
		}
	}
}
```

- Laravelはカスタムネームスペースを完全にはサポートしていない
	- `IlluminateFoundationApplication`クラスで`App/`を探すようにハードコーティングされている
	- 自作する必要がある

```php
namespace App;

class Application extends \Illuminate\Foundation\Application
{
	protected $namespace = 'App\\';
}
```

`bootstrap/app.php`を上書きする

```php
use App\Application;

$app = (new Application(
	$_ENV['APP_BASE_PATH'] ?? dirname(__DIR__)
))->useAppPath('src/App');
```

---

- 同じ技術的特性をもつコードのグループではなく、関連するビジネスコンセプトのグループで考えるようにすることが重要

### データを扱う

- コードをドメインでグループ化する
	- 重要な基盤となるコンセプト
- コントローラやジョブの構築からではなく、Modelから構築を始める
- シンプルデータ指向パターンを理解するために型に触れるべき

### 型理論

- 強い型と弱い型
	- 強い型は一度代入が行われると、再代入で異なる型は含められない
	- PHPは弱い型
- 完全に制御するには `declare(strict_types=1)`が必要になる
	- しかしPHPの型システムは弱いので、型ヒントはその時点の変数の型を保証するだけ
	- 将来もつかもしれない値については担保しない
	- PHPは入力が文字列から始まるので理にかなっている
- 強い型はプログラマにコードが実際に動作することを保証してくれる
	- バグがないコードを書くことは不可能だが、型付けが強ければコンパイル時点ではじき出せる
- 強力な型システムはコードを書く時にプログラムを実行するのではなく、洞察を得ることができる
- 静的型システムと動的型システム

### 非構造化データを構築する

- PHPには構造体がない
	- クラスとオブジェクトで代用が可能

```php
class CustomerData { 

 public string $name;
 public string $email;
 public Carbon $birth_date; 

}
```

```php
function store(CustomerRequest $request, Customer $customer)
{ 

 $validated = CustomerData::fromRequest($request); 

 $customer->name = $validated->name;
    $customer->email = $validated->email;
    $customer->birth_date = $validated->birth_date; 

// ...
}
```

- 非構造化データを型で包んで、確実に利用できるパターンを Data Transfer Objectsと呼ぶ
	- 平均以上のLaravelプロジェクトで使うべき
	- 認知負荷も減らす
- DTOをリクエストデータにマッピングする必要がある

### DTO factories

アプリケーション層に置く

```php
 
class CustomerDataFactory
{
	public function fromRequest(
		CustomerRequest $request
	): CustomerData {
		return new CustomerData([
			'name' => $request->get('name'),
			'email' => $request->get('email'),
			'birth_date' => Carbon::make(
				$request->get('birth_date')
			),
		]);
	}
}

```

```php
 
// このアプローチは間違っている　が、実用的なため採用している(今は8で名前付きパラメータのほうがBetter)
use Spatie\DataTransferObject\DataTransferObject; 

class CustomerData extends DataTransferObject { 

// ...

 public static function fromRequest(
 	CustomerRequest $request 
 ): self {
 return new self([ 
 	'name' => $request->get('name'),
 	'email' => $request->get('email'),
	 'birth_date' => Carbon::make( 
 			$request->get('birth_date')
			), 
		]);
 	}
}
```

- ドメイン内にアプリケーション固有のコードを混在させることは最良のアイデアではない
	- 外部からデータを扱う際は、DTOに変換する必要がある
	- PHP自身が名前付きパラメータをサポートしていない（8からはしてる？）
- DTOが各プロパティに個別のパラメータを持つコンストラクタを持つのは好ましくない
	- スケールしない
	- NULL値やデフォルト値のプロパティを扱う時に混乱する
	- DTOに配列を渡し、その配列内のデータに基づいてDTOを構築させる方法が望ましい

named parametersをサポートしているなら

```php
public function fromRequest(
	CustomerRequest $request
): CustomerData {
	return new CustomerData(
		name: $request->get('name'),
		email: $request->get('email'),
		birth_date: Carbon::make(
			$request->get('birth_date')
		),
	);
}
```

### 型付きプロパティの代替案

型付きプロパティを使うという方法

```php
use Spatie\DataTransferObject\DataTransferObject; 

class CustomerData extends DataTransferObject { 

	/** @var string */
	public $name; 

	/** @var string */
	public $email; 

	/** @var \Carbon\Carbon */ 
	public $birth_date;
}
```

- DocBlockはデータがその型であることは保証してくれない

### PHP8のDTO Memo

```php
class CustomerData
{
	public function __construct(
		public string $name,
		public string $email,
		public Carbon $birth_date,
	) {}
}

$data = new CustomerData(...$customerRequest->validated());

```

## Actions

- ビジネスの機能をランダムな関数やクラスに分散させたくない

ex) 管理者が請求書を作成する　という要件はDBに請求書を保存する以外にもやることがある

- 請求書番号の生成
- 請求書をデータベースに保存
- 支払いサービスを通して支払いを作成する
- 関連する情報をまとめたPDF作成
- PDFをお客に送信する

- Laravelではこれを処理するFat Modelを作成するのが一般的である
- これをアクションとして扱う

### 用語

- アクションの構成について
	- Domainに存在する必要がある
	- 抽象化やインターフェースを持たない単純なクラスである
	- アクションは入力を受けて、何かを行い、出力を行うクラスであること
	- つまり、public methodとコンストラクターだけ
- 例を実現すると `CreateInvoiceAction`となる
	- 他にも長い名前をつける必要がある。短い名前よりもクラスが何であるか明確にすることが最も重要である

```php

class CreateinvoiceAction
{
	public function __invoke(InvoiceData $invoiceData): Invoice
	{
		// ...
	}
}

```

他のアクションからアクションを構成することもある

```php
class CreateinvoiceAction
{
	private CreateInvoiceLineAction $createinvoiceLineAction;
	
	public function __construct(
		CreateInvoiceLineAction $create
	) {
		/* ...*/
	}
	
	public function __invoke(InvoiceData $invoiceData): Invoice
	{
		foreach ($invoiceData->lines as $lineData) {
			$invoice->addLine(
				($this->createInvoiceLineAction)($lineData)
			);
		}
	}
}
```

- メソッドインジェクションよりコンストラクタインジェクションを押す(この講座は)
- 命名としてhandleは良くない。最終的にはexecuteに落ち着いた
	- ただ、重要なのは名前よりもアクションの使い方パターンなので独自の命名規則を考えるのは自由

### 実践

- 再利用性
	- アクションを再利用できるように十分に小さく分割する
	- しかし、クラスの過負荷にならないように十分な大きさにしておく

ex
- 請求書を作成する
- 請求書をプレビューする
- 2つのコントローラが必要になる。しかし請求書に基づいてPDFを生成することはどちらでも行われる

- コードを抽象化しすぎないように気をつける
	- 時にはコードをコピペしたほうがよい場合もある
- 抽象化を行う時の経験則は「コードの技術的な特性」よりも「機能」について考えることが大切
	- 似たようなことをしていても、異なる文脈で使われる場合は抽象化を早い段階では行うべきでない

- アクションの大半はユーザーストーリーに特化したものであり、再利用できない
	- 最も重要なのは、技術的利点ではなく、「より現実の世界に近い方法で考えることができるようになること」
	- アクションはシステムによってもたらされる認知的負荷を軽減する
		- 請求書がどのように作成されるかについて作業する必要がある場合、アクションクラスに移動し、そこから始めるだけで良い

CreateInvoiceLineActionを例に

- どの製品に請求がくるかデータを取る
- 量と期間を受け取る
- 合計金額を計算する
- 値段と税別値段を計算する

- 堅牢でシンプルなユニットテストを書くことができる
- アクションが適切にユニットテストされていればアプリの機能の大部分が意図的に動作すると確信できる

### アクションを構成する

- アクションの重要な特徴
	- 依存性をコンストラクタでコンテナからデータを渡し、executeメソッドでコンテキスト関連のデータを渡す
	- アクションからアクション、アクションから・・・と自由にできる
	- しかし依存関係の連鎖が深いとコードが複雑になり、互いに強く依存し合う
- 例でいうと `CreateInvoiceLineAction`にはVAT価格の計算のような詳細までは関わらせたくない
	- VatCalculatorをSupportに作る

```php

class CreateInvoiceLineAction
{
	private VatCalculator $vatCalculator;
	
	public function __construct(VatCalculator $vatCalculator)
	{
		$this->vatCalculator = $vatCalculator;
	}
	
	public function execute(
		InvoiceLineData $invoiceLineData
	): InvoiceLine {
		// ...
	}
}

```
以下のように使う
```php
public function execute(
	InvoiceLineData $invoiceLineData
): InvoiceLine {
	$item = $invoiceLineData->item;
	
	if ($item->vatIncluded()) {
		[$priceIncVat, $priceExclVat] =
			$this->vatCalculator->vatIncluded(
				$item->getPrice(),
				$item->getVatPercentage()
			);
	} else {
		[$priceIncVat, $priceExclVat] =
			$this->vatCalculator->vatExcluded(
				$item->getPrice(),
				$item->getVatPercentage()
			);
	}
	
	$amount = $invoiceLineData->item_amount;
	
	return new InvoiceLine([
		'item_price' => $item->getPrice(),
		'total_price' => $amount * $priceInVat,
		'total_price_excluding_vat' => $amount * $priceExclVat,
	]);
}

```

- コンポジションによって、個々のアクションを小さく保ちながら、複雑なビジネス機能を明確かつ保守可能な方法でコーディングする

### アクションの代替

- アクションを必要としないパラダイム
	- DDDでよく聞くコマンドとハンドラ
		- アクションはこれらを簡略化したもの
		- コマンドバスのほうがアクションより柔軟性があるのは事実だが、コード量が増える
		- アクションをコマンドとハンドラに分割するのはOver Power
	- イベントドリブンシステム
		- より柔軟性がある
		- これもまたOver Powerになりがち
- アーキテクチャを選定する上で、プロジェクトの具体的なニーズに目を向けることが重要になる
	- アクションは柔軟性・再利用性・認知負荷の軽減に優れている

## Models

- ある程度Eloquentを諦める必要がある

### Models != business logic

```php
class Invoice extends Model
{
	public function getTotalPriceAttribute(): int
	{
		return $this->invoiceLines
			->reduce(
				fn(int $totalPrice, InvoiceLine $invoiceLine) =>
					$totalPrice + $invoiceLine->total_price,
					0
			);
	}
}
```

```php
class InvoiceLine extends Model
{
	public function getTotalPriceAttribute(): int
	{
		$vatCalculator = app(VatCaluculator::class);
		
		$price = $this->item_amount * $this->item_price;
		
		if ($this->price_excluding_vat) {
			$price = $vatCalculator->totalPrice(
				$price,
				$this->vat_percentage
			);
		}
		
		return $price;
	}
}

```

請求書の合計金額を計算しているが、これはアクションによって表現されるべきユーザーストーリー

- 単一責任によってクラスがいかに小さく保たれ、保守しやすく、テストしやすいか、依存性注入がいかにサービスロケーションより優れているかという議論を始めることができる
- 何百行ものコードを持つモデルクラスは保守性を維持することが出来ない
	- モデルやその目的はデータを提供するだけで、そのデータが正しく計算されているかどうかは別のものに任せておけば良い

### モデルの縮小化

- 理想形
	- 事前に計算できないためのかんたんなアクセサ、キャスト、リレーションだけ残したい
- query scopesはquery builder classに移動させることができる
- Query BUilder CLassはEloquentの通常の使用方法で、scopeはその上にある単なる構文上のシンタックスシュガー

クエリビルダークラス
```php
namespace Domain\Invoices\QueryBuilders;

use Domain\Invoices\States\Paid;
use Illuminate\Database\Eloquent\Builder;

class InvoiceQueryBuilder extends Builder
{
	public function wherePaid(): self
	{
		return $this->whereState('status', Paid::class);
	}
}
```

override the newEloquentBuilder method in model
```php
namespace Domain\Invoices\Modesl;

use Domain\Invoices\QueryBuilders\InvoiceQueryBuilder;

class Invoice extends Model
{
	public function newEloquentBuilder($query): InvoiceQueryBuilder
	{
		return new InvoiceQueryBuilder($query);
	}
}

```

- フレームワークを受け入れるという意味で、リポジトリのような新しいパターンを導入する必要がない。
- フレームワークが提供するコモディティを使いながら、特定の場所でコードが大きくなりすぎないようにバランスを取っている
- この考え方を使って、リレーションのためのカスタムコレクションクラスを提供することも可能

カスタムコレクションクラスの例

```php
namespace Domain\Invoices\Collections;

use Domain\Invoices\Models\InvoiceLines;
use Illuminate\Database\Eloquent\Collection;

class invoiceLineCollection extends Collection
{
	public function creditLines(): self
	{
		return $this->filter(fn (InvoiceLine $invoiceLine) => 
			$invoiceLine->isCreditLine()
		);
	}
}
```
Modelと連結させる

```php
namespace Domain\Invoices\Models;

use Domain\Invoices\Collection\InvoiceLineCollection;

class InvoiceLine extends Model
{
	public function newCollection(
		array $models = []
	): InvoiceLineCollection {
		return new InvoiceLineCollection($models);
	}
	
	public function isCreditLine(): bool
	{
		return $this->price < 0.0;
	}
}

```

InvoiceLineに対してHasManyリレーションを持つモデルはコレクションクラスを使用するようになった

```php
$invoice
	->invoiceLines
	->creditLines()
	->map(function (InvoiceLine $invoiceLine) {
		// ...
	});
```

ビジネスロジックを提供させるのではなく、クリーンでデータ指向のモデルを維持するようにする。