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

### イベント駆動型モデル

- 複雑さを犠牲にして柔軟なシステムを提供することができる

```php
class Invoice
{
	protected $dispatchesEvents = [
		'saving' => InvoiceSavingEvent::class,
		'deleting' => InvoiceDeletingEvent::class,
	];
}

```

```php
class InvoiceSavingEvent
{
	public Invoice $invoice;
	
	public function __construct(Invoice $invoice)
	{
		$this->invoice = $invoice;
	}
}

```

```php
use Illuminate\Events\Dispatcher;

class InvoiceSubscriber
{
	private CalculateTotalPrinceAction $calculateTotalPrinceAction;
	
	public function __construct(
		CalculateTotalPriceAction $calculateTotalPrinceAction
	) { /* ... */}
	
	public function saving(InvoiceSavingEvent $event): void
	{
		$invoice = $event->invoice;
		
		$invoice->total_price =
			($this->calculateTotalPriceAction)($invoice);
	}
	
	public function subscribe(Dispatcher $dispatcher): void
	{
		$dispatcher->listen(
			InvoiceSavingEvent::class,
			self::class . '@saving'
		);
	}
}


```

```php
class EventServiceProvider extends ServiceProvider
{
	protected $subscribe = [
		InvoiceSubscriber::class,
	];
}
```

- このアプローチだと独自のカスタムモデルイベントにフックし、Eloquentイベントと同じように処理する柔軟性がある
- Subscriberクラスによってモデルは小さく維持され、保守しやすくなる

### Empty bags of nothingness

- Martin Fowlerは「オブジェクトが単なるデータの空っぽの袋にならないようにする必要がある」と書いた
	- アンチパターンとして
- モデルは「単なるデータの空っぽの袋」に見えるが、そうではない
	- accessors, castsによってモデルはデータベース内のプレーンなデータと開発者が使いたいデータとの間にリッチなレイヤーを提供する
- Alan Kayのビジョン
	- 「プロセス指向」ではなく「オブジェクト指向」と呼んだことを後悔している
	- プロセスとデータを分割する推進派だと主張する
- 自分自身が色々な設計に挑戦し、問題を解決するために最適な方法を考えることに意味がある

## States

Stateパターンはモデルに状態固有の振る舞いを追加し、かつモデルをキレイに保つための最良の方法の1つ

モデルクラスがビジネスロジックを処理しないようにすることで管理しやすくすることを目的としている。

### The state pattern

請求書の例

素朴なFat Model Approch
```php
class Invoice extends Model
{
	// ...
	
	public function getStateColour(): string
	{
		if ($this->state->equals(InvoiceState::PENDING())) {
			return 'orange';
		}
		
		if ($this->state->equals(InvoiceState::PAID())) {
			return 'green';
		}
		
		return 'gray';
	}
}
```

状態値を表現するためにある種のenumクラスを使っている。enumを使ってカプセル化できる

```php
class InvoiceState extends Enum
{
	private const PENDING = 'pending';
	private const PAID = 'paid';
	
	public function getColour(): string
	{
		if ($this->value === self::PENDING) {
			return 'orange;'
		}
		
		if ($this->value === self::PAID){
			return 'green';
		}
		
		return 'gray';
	}
}

```
Modelは以下のように
```php
class Invoice extends Model
{
	// ...
	
	public function getStateColour(): string
	{
		return $this->state->getColour();
	}
}
```

配列を使えばもっと短く書くこともできる
```php
class InvoiceState extends Enum
{
	public function getColour(): string
	{
		return [
			self::PENDING => 'orange',
			self::PAID => 'green',
		][$this->value] ?? 'grey';
	}
}
```

PHP8だとmatchを使える

```php
class InvoiceState extends Enum
{
	public function getColour(): string
	{
		return match($this->value) {
			self::PENDING => 'orange',
			self::PAID => 'green',
			default => 'grey',
		};
	}
}
```

- このアプローチを使うと、モデルかenumクラスのどちらかに特定の状態が何をすべきかを知っていなければならない
	- 状態がどのように機能するかを知って無ければいけないという責任を追加することになる

抽象クラスInvoiceStateから始める
```php
abstract class InvoiceState
{
	abstract public function colour(): string;
}

class PendingInvoiceState extends InvoiceState
{
	public function colour(): string
	{
		return 'orange';
	}
}

class PaidInvoiceState extends InvoiceState
{
	public function colour(): string
	{
		return 'green';
	}
}
```

かんたんにユニットテストが可能

```php
class InvoiceStateTest extends TestCase
{
	public function the_colour_of_pending_is_orange
	{
		$state = new PendingInvoiceState();
		
		$this->assertEquals('orange', $state->colour());
	}
}
```

- 複雑なルールを実現するには足りないものがある
	- モデルを参照できるようにする必要がある
	- Invoiceモデルをstateクラスにインジェクトする必要がある

```php
abstract class InvoiceState
{
	protected $invoice;
	
	public function __construct(Invoice $invoice) { /* ... */}
	abstract public function mustBePaid(): bool;
}
```

```php
class PendingInvoiceState extends InvoiceState
{
	public function mustBePaid(): bool
	{
		return $this->invoice->total_price > 0
			&& $this->invoice->type->equals(InvoiceType::DEBIT());
	}
}

class PaidInvoiceState extends InvoiceState
{
	public function mustBePaid(): bool
	{
		return false;
	}
}
```

かんたんにユニットテストを書くことができ、単純に以下のことができる
```php
class Invoice extends Model
{
	public function getStateAttribute(): InvoiceState
	{
		return new $this->state_class($this);
	}
	
	public function mustbePaid(): bool
	{
		return $this->state->mustBePaid();
	}
}

```

- `state_class` に具象モデルの状態のクラスを保存する
- `spatie/laravel-model-states` package
	- PHP8.1, Laravel 8.7系で解決されてそう

- State Patternは解決策の半分にすぎない
- 状態の遷移について見てみる必要がある

### Transitions

- ビジネスロジックをモデルから遠ざけ、データベースから実行可能な方法でデータを提供する
	- 同じ考え方を状態や遷移に適用することができる
- Stateを使用する際には、データベースに変更を加えたり、メールを送ったりなどのSide Effectを避ける必要がある
	- Stateはデータの読み取りや提供のために使用されるべき
- Transitionは何も提供しない
	- モデルの状態が次から次へと正しく遷移することを確認する
- この2つの問題を別々のクラスに分割することで、テスト容易性・認知負荷を軽減する
- Transitionとはモデルを受け取り、その状態を別のものに変更するためのクラスである
	- 場合によってはログを買いたり、状態遷移に関する通知を送る副作用がある場合もある

```php
class PendingToPaidTransition
{
	public function __invoke(Invoice $invoice): Invoice
	{
		if (! $invoice->mustBePaid()) {
			throw new InvalidTransitionException(self::class, $invoice);
		}
		
		$invoice->status_class = PaidInvoiceState::class;
		$invoice->save();
		
		History::log($invoice, "Pending to Paid");
	}
}
```

- モデル上で許容されるすべての遷移を定義する
- フードの下にあるTransitionクラスを使用して、状態を直接別の状態に遷移させる
- 一連のパラメータを元に、どのような状態に遷移するかを自動的に決定する


### transitionsがないState

- 遷移がない状態も存在する

```php
class PendingInvoiceState extends InvoiceState
{
	public function mustBePaid(): bool
	{
		return $this->invoice->total_price > 0
			&& $this->invoice->type->equals(InvoiceType::DEBIT());
	}
}
```

InvoiceTypeもまたステートパターンである。
InvoiceTypeは変更されることはない

```php
abstract class InvoiceType
{
	protected Invoice $invoice;
	
	// ...
	
	abstract public function mustBePaid(): boo;
}

class CreditInvoiceType extends InvoiceType
{
	public function mustBePaid(): bool
	{
		return false;
	}
}

class DebitInvoiceType extends InvoiceType
{
	public function mustBePaid(): bool
	{
		return true;
	}
}
```

`PendingInvoiceState`のリファクタリングができる

```php
class PendingInvoiceState extends InvoiceState
{
	public function mustbePaid(): bool
	{
		return $this->invoice->total_price > 0
			&& $this->invoice->type->mustBePaid();
	}
}

```

- コードの中のif/else文を減らすことで、コードは直線的になる

## Enums

### EnumsかStatesか

- enumはstateとは何が違うのか？いつenumsを使用するのか？
- enumとstateパターンの両方を使って同じ結果を得ることができるので厄介
- enumとstateの実装の違いは、stateパターンでは可能な値毎に専用のクラスを用意することで、それらの条件を排除している点
	- state パターンの目的は条件式をすべて取り除き、代わりにポリモーフィズムの力を借りてプログラムの流れを決定する
	- 選択する1つの方法として、コード中の条件分岐をなくすためにstateパターンを使い、それ以外はすべてenumsを使うと主張できる
	- じゃあ「それ以外」は何？
		- stateパターンでモデリングできるような条件
	- 現実主義的な考え方で行くと、stateパターンは大きなオーバーヘッドがある
		- つまり、Enumが使える余地がある
	- 関連する値のコレクションが必要で、アプリケーションの流れが実査いにその値によって決定される部分が少ないのであれば、Enumを使うことができる
	- より多くの値に関連する機能を列挙している事に気づいたらstateパターンを検討し始める時
	- CaseByCaseである

### Enums!
PHPではEnumがない！（8.1でsolve）

`myclabs/php-enum`で代用する
```php
class Invoice
{
	public function setType(InvoiceType $type): void
	{
		$this->type = $type;
	}
}
```

定数をprivateにして、マジックメソッドを使ってその値を解決する
```php
use MyClabs\Enum\Enum;

class InvoiceType extends Enum
{
	private const CREDIT = 'credit';
	private const DEBIT = 'debit';
}

$invoice->setType(InvoiceType::CREDIT());
```

- `__callStatic()`を使ってcreditを取得している
- しかし、`__callStatic()`に依存することで自動補完やリファクタリングのような静的解析の利点を失っている
	- DocBlockで解決できる

```php
use MyClabs\Enum\Enum;

/**
 * @method static self CREDIT()
 * @method static self DEBIT()
 */
class InvoiceType extends Enum
{
	private const CREDIT = 'credit';
	private const DEBIT = 'debit';
}

$invoice->setType(InvoiceType::CREDIT());

```

## Managing domains

### TeamWork

- 新しいディレクトリ構造や複雑な原理の使用で、新しい開発者がすぐに参加するのは難しい?
- コードが技術的にどのように構成されているかではなく、理解すべき膨大な量のビジネス知識が大変
	- ビジネスを理解するのに時間がかかる
- 混乱する余地をできる限り少なくし、プロジェクトを長く健全に保つことが大切
- アーキテクチャがすべてのプロジェクトの特効薬になるわけではない


### ドメインの特定

- ビジネスの問題を理解し、それをコードに変換すること
	- コードはあくまで手段であり、問題解決にフォーカスする
- クライアントと対面する時間を確保すること。動作するプログラムを書くために必要な知識を引き出すには時間がかかる

- 「コードを書くプログラマー」ではなく、「現実の問題と技術的な解決策の翻訳者」
	- 長期的なプロジェクトに携わる上で重要

- 時間の経過とともに変化するDomainグループを恐れる必要はない
- ドメイン構造を繰り返しリファクタリングし続ける
	- ドメインは後でいつでもリファクタリングができるので怖がらずに使い始める
- クライアントと共に変化するものを理解する必要がある

## Testing Domain

### Test Factories

#### FactoryのBad Point

- `$factory`に対してIDEはどのようなオブジェクトなのか全く理解していない
- `states()`は文字列として定義されるので、ブラックボックスになる
- factoryのresultにType Hintingがない
- 大きなドメインがある場合、テストスイートには数個以上必要な場合があり、時間経過とともに管理が難しくなる
- DTOやRequestに対応していない
- Factoryクラスの目標は、システムのセットアップにあまり時間をかけずに統合テストを書くのを助けること
	- 「単体テスト」ではなく「統合テスト」
- ビジネスロジックをテストする、ということはクラスの孤立した部分をテストするのではなく、データベースにデータが存在することを必要とする、複雑で入り組んだビジネスルールをテストすることを意味する。

### A Basic Factory

```php
class InvoiceFactory
{
	public static function new(): self
	{
		return new self();
	}
	
	public function create(array $extra = []): Invoice
	{
		return Invoice::create(array_merge(
			[
				'number' => 'I-1',
				'status' => PendingInvoiceState::class,
				// ...
			],
			$extra
		));
	}
}

```

#### Factories in factories

```php
class InvoiceFactory
{
	private ?string $status = null;
	
	public function create(array $extra = []): Invoice
	{
		$invoice = Invoice::create(array_merge(
			[
				'status' => $this->status ?? PendingInvoiceState::class
			],
			$extra
		));
		
		if ($invoice->status->isPaid()) {
			PaymentFactory::new()->forInvoice($invoice)->create();
		}
		
		return $invoice;
	}
	
	public function paid(): self
	{
		$clone = clone $this;
		
		$clone->status = PaidInvoiceState::class;
		
		return $clone;
	}
}

```

- InvoiceFactoryに直接設定を渡しすぎるとすぐにごちゃごちゃになってしまう
- InvoiceFactoryにオプジョンでPaymentFactoryを渡すことで、InvoiceFactoryの外から好きなようにPaymentFactoryを設定できるようにする

```php
public function paid(PaymentFactory $paymentFactory = null): self
{
	$clone = clone $this;
	
	$clone->status = PaidInvoiceState::class;
	$clone->paymentFactory = $paymentFactory ?? PaymentFactory::new();
	
	return $clone;
}

```

`create`メソッド

```php
if ($this->paymentFactory) {
	$this->paymentFactory->forInvoice($invoice)->create();
}
```

example
```php
public function test_case()
{
	$invoice = InvoiceFactory::new()
		->paid(
			PaymentFactory::new()->type(VisaPaymentType::class)
		)
		->create();
}
```

```php
public function test_case()
{
	$invoice = InvoiceFactory::new()
		->expiresAt('2020-01-01')
		->paid(
			PaymentFactory::new()->paidAt('2020-05-20')
		)
		->create();
}

```

### Immutable factories

- 既にあるものを使いまわしたいケースもある

```php
$invoiceFactory = InvoiceFactory::new()
	->expiresAt(Carbon::make('2020-01-01'));

$invoiceA = $invoiceFactory->paid()->create();
$invoiceB = $invoiceFactory->create();
```

baseとなるFactoryを設定し、それをテスト全体で再利用すれば予期せぬ副作用を心配する必要はない

#### 原則

- Factoryの中にFactoryを構成する
- FactoryをImmutableにする

欠点としてはオーバーヘッドが大きくなる。しかし総コストは落ちるし、抽象クラスで少し負担を減らせる

```php
abstract class Factory
{
	abstract public function create(array $extra = []);
	
	public function times(int $times, array $extra = []): Collection
	{
		return collect()
			->times($times)
			->map(fn() => $this->create($extra));
	}
}
```

DTOにも使えるし、リクエストクラスにも使うことができる

### Testing DTOs

- DTOが強く型付けされた方法でデータを表現するだけであれば何もテストは必要ない
- 手動でマッピングする場合はテストを書く必要がある

```php
/** @test */
public function form_booking_store_request()
{
	$unit = UnitFactory::new()->create();
	
	$dto = BookingData::fromStoreRequest(new BookingStoreRequest([
		'name' => 'test',
		'unit_id' => $unit->id,
		'date_start' => '2020-12-01',
		'date_end' => '2020-12-05',
	]));
	
	$this->assertInstanceOf(BookingData::class, $dto);
}
```

他は型チェックでカバーできるので、オブジェクトであるかどうかのアサートだけ行う

exceptions Test
```php
/** @test */
public function from_booking_store_request_without_unit_fails()
{
	$this->expectException(ModelNotFoundException::class);
	
	BookingData::fromStoreRequest(new BookingStoreRequest([
		'name' => 'test',
		'date_start' => '2020-12-01',
		'date_end' => '2020-12-05',
	]));
}
```

### Testing actions

```php
class CreateInvoiceAction
{
	private CreateInvoiceLineAction $createInvoiceLineAction;
	
	public function __construct(
		CreateInvoiceLineAction $createInvoiceLineAction
	) { /* ... */ }
	
	public function execute(
		InvoiceData $invoiceData
	): Invoice { /* ... */ }
}
```

Actionのテストは
- やるべきことをやっているかどうか
- その元となるアクションを正しい方法で使っているか
の上記2点が重要になる
つまり、CreateInvoiceLineActionは内部的に行うべきこと、つまり新しいInvoiceLineをそのすべてのプロパティとともに保存するかどうかはテストしない

```php
/* @test */
public function invoice_is_saved_in_the_database()
{
	$invoiceData = InvoiceDataFactory::new()
		->addInvoiceLineDataFactory(
			InvoiceLineDataFactory::new()
				->withDescription('Line A')
				->withItemAmount(1)
				->withItemPrice(10_00)
		)
		->addInvoiceLineDataFactory(
			InvoiceLineDataFactory::new()
				->withDescription('Line B')
				->withItemAmount(3)
				->withItemPrice(33_00)
		)
		->create();
	
	$action = app(CreateInvoiceAction::class); // コンテナから解決
	
	$invoice = $action->execute($invoiceData);
	
	$this->assertDatabaseHas($invoice->getTable(), [
		'id' => $invoice->id,
	]);
	
	$this->assertNotNull($invoice->number);
	
	$expectedTotalPrice = 1 * 10_00 + 3 * 33_00;
	
	$this->assertEquals(
		$expectedTotalPrice,
		$invoice->total_price
	);
	
	$this->assertCount(2, $invoice->invoiceLines);
}
```

- アサーションを別々のテストメソッドに分割してもOK
- セットアップ部分をテストクラスのsetUpに移動してもOK

PDFを生成するようなテストを遅くするものは別Actionに分離され、テストの際はそのアクションをMockすることによってテストを実現する

```php
namespace Tests\Mocks\Actions;

use Domain\Pdf\Actions\GeneratePdfAction;

class MockGeneratePdfAction extends GeneratePdfAction
{
	public static function setUp(): void
	{
		app()->singleton(
			GeneratePdfAction::class,
			fn () => new self()
		);
	}
	
	public function execute(ToPdf $toPdf): void
	{
		return;
	}
}
```

### Testing models

```php
public function where_active()
{
	$factory = UnitFactory::new();
	
	$activeUnit = $factory->active()->create();
	
	$inactiveUnit = $factory->inactive()->create();
	
	$this->assertEquals(
		1,
		Unit::query()
			->whereActive()
			->whereKey($activeUnit->id)
			->count()
	);
	
	$this->assertEquals(
		0,
		Unit::query()
			->whereActive()
			->whereKey($inactiveUnit->id)
			->count()
	);
}
```

#### Susscriberのテスト

```php
public function test_saving()
{
	$subscriber = app(InvoiceSubscriber::class);
	
	$event = new InvoiceSavingEvent(InvoiceFactory::new()->create());
	
	$subscriber->saving($event);
	
	$invoice = $event->invoice;
	
	$this->assertNotNull($invoice->total_price);
}
```

## Entering the application layer

- コードベースにおける目的ではなく、実世界で何に似ているかを基準にコードをグループ化すること
- アプリケーション層には何が属するのか？
- どのようにコードをグループ化するのか？

### Several applications

- HTTP関連アプリ開発がメインになる
	- Controllers
	- Requests
	- Request-specific validation rules
	- Middleware
	- Resources
	- ViewModels
	- HTTP QueryBuilders (the ones that parse URL queris)
- アプリケーションの目的は、何らかの入力を得て、それをドメイン層で使えるように変換し、出力をユーザーに示すか、どこかに保存すること

### Structuring HTTP applications

LaravelはデフォルトでクラシックHTTPアプリケーションで行うことを推奨している

App/Admin
- Http
	- Controllers
	- Kernel.php
	- Middleware
- Requests
- Resources
- Rules
- ViewModels

この構造は小規模なプロジェクトでは問題ないが、スケールすることができない。
問題の核心として、コントローラにはコントローラ、リクエストにはリクエスト、ビューモデルにはビューモデルというように実際の意味ではなく、技術的なプロパティに基づいてコードをグループ化している

- 解決策は、ドメインの場合と同じで「一緒になるべきコードをまとめる」こと。
	- 「アプリケーション・モジュール」、略して「モジュール」と呼ぶ
	- モジュールはドメイン上に一対一でマッピングされるべき？
		- 重複していてもかまわないが、必須ではない
	- Settingsモジュールがあり、複数のドメイングループに一度触れている。
		- そのSettingsモジュール自体で一つの機能であり、そのコードはグループされるべきである

Supportはグローバルにアクセス可能なすべてのコードを保持し、フレームワークまたはスタンドアロンパッケージの一部であるようにする。リクエストのベースとなるクラスや、あらゆる場所で使われるミドルウェアなどが入る。

## View Models