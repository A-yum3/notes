#php/pest #source/video

### HTTP通信のスタブはHTTP Facadeを使う(Laravelの)

```php
HTTP::fake(['https://example.com/*' => response()])
```

### PestのTime Pluginを使う
```php
testTime()->freeze()
	
testTime()->subSecond();
```

### お手製Mock

- tests/fakesディレクトリを切る
- モック対象をextendsしFake Classを作成、setUpで`app()->instance(xxx::class, new self());
- モック対象メソッドをoverrideする
- expectMethodをFakeの中に書く

### Middleware Test

```php
Route::get('my-test-route', fn() => 'ok')
	->middleware(RedirectMiddleware:: class)

get('/')->assertStatus(200);

Redirect::factory()->create([
	'from' => '/',
	'to' => '/new-homepage'
]);

get('/')->assertRedirect('/new-homepage'); 
```

### Validation Test

Errorが出るものを指定する
例えば、title => required, stringだったら、データがなかった時にerrorが出るので、assertSessionHasErrors(['title'])とすればAssertする
[assertSessionHasErrors](https://readouble.com/laravel/8.x/ja/http-tests.html#assert-session-has-errors)

```php
$post = Post::factory()->create();

post(action([BlogPostAdminController::class, 'update'], $post->slug), [
])
	->assertSessionHasErrors(['title', 'author', 'body', 'date']);
```

`dumpSession()` を使ってセッション情報をdumpし、`assertSessionHasErrors`の引数arrayに連想配列でkey とメッセージを指定すると明示的になる。

```php
post(action([BlogPostAdminController::class, 'update'], $post->slug), [
	'title' => $post->title
	'author' => $post->author,
	'body' => $post->body,
	'date' => '01/01/2021',
])->dumpSession()->assertSessionHasErrors([
	'date' => 'The date does not match the format Y-m-d.'
])
```

beforeEachで$thisが使える

### DataProvider

```php
it('xxx', function(string $value, bool $expectedResult) {
	$result = (new UppercaseRule())->passes('name', $value);
	expect($result)->toBe($expectedResult);
})->with([
	['MY STRING', true],
	['my string', false],
	['My string', false],
])
```

### Image Upload Test

```php
it('xxx', function() {
	testTime()->freeze('2021-01-01 00:00:00');
	
	Storage::fake('public');
	
	$file = UploadedFile::fake()->image('test.jpg');
	
	post(action(UploadController::class), [
		'file' => $file
	]);
	
	Storage::dist('public')->assertExists('test.jpg');
})
```

### Json Test

$thisを使えばimportなしで使えるが、importしたほうがエレガントになる

```php
it('xxx', function() {
	$post = BlogPost::factory()->count(5)->create();
	
	$returnedPosts = getJson(action([BlogPostController::class, 'index']))
		->assertSuccessful()
		->assertJsonCount(5, 'data')
		->assertJson(function(AssertableJson $json) {
			$json->has('data', 5)
				->has('data.0', function(AssertableJson $json) {
					$json
						->where('id', $posts[0]->id)
						->where('slug', $posts[0]->slug)
						->has('date', $posts[0]->date->toDateTimeLocalString)
						->whereAllType([
							'id' => 'integer',
							'slug' => 'string',
							'date' => 'string',
						])
						->etc();
				})
		})	
		
});
```

```php
it('xxx', function() {
	$post = BlogPost::factory()->count(5)->create();
	
	$returnedPosts = getJson(action([BlogPostController::class, 'index']))
		->assertSuccessful()
		->assertjsonPath('data.title', $post->title)
```

### dataset

```php
it('xxx' function(Closure $request) {
})->with('requests');


dataset('requests', [
	fn(BlogPost $post) => get(action([BlogPostAdminController::class, 'index'])),
])
```

### Job Testing

`Bus::assertDispatched`

```php
it('xxx', function() {
	BUS::fake();
	
	BlogPost::factory()->create();
	
	Bus::assertDispatched(CreatedogImageJob::class, 1);
})
```

### Snapshots Test

`composer require spatie/pest-plugin-snapshots --dev`

今あるtextデータから一致しているかどうかを判定する

### Coverage

`./vendor/bin/pest --coverage`

PHPStormなら右クリで簡単にできる

### Custom Expectations

`extend`を使い、Custom Expectationsを作る

```php
expect(5)->toBeInTheRange(1, 10);

expect()->extend('toBeInTheRange', function(int $min, int $max) {
	return $this
		->toBeGreaterThanOrEqual($min)
		->toBeLessThanOrEqual($max);
});
```

### Exceptions

```php
it('xxx', function() {
	$post = Post::factory()->published()->create();
	
	$post->publish();
})->throws(BlogPostCouldNotBePublished::class, 'message
```

### Custom Factory

RequestデータやJsonカラムなどは差し込みが大変なのでカスタムFactoryを作る

```php

?php

namespace Tests\Factories;

use App\Models\BlogPost;

use Carbon\Carbon;

class BlogPostRequestDataFactory

{

private string $title = 'Title';

private string $author = 'Author';

private string $body = 'Body';

private string $date = '2021-01-01';

public static function new(): self

{

return new self();

}

public function create(array $extra = []): array

{

return $extra + [

'title' => $this->title,

'author' => $this->author,

'body' => $this->body,

'date' => $this->date,

];

}

public function withTitle(string $title): self

{

$clone = clone $this;

$clone->title = $title;

return $clone;

}

public function withDate(string|Carbon $date): self

{

$clone = clone $this;

$clone->date = $date instanceof Carbon

? $date->format('Y-m-d')

: $date;

return $clone;

}

public function withPost(BlogPost $post): self

{

$clone = clone $this;

$this->title = $post->title;

$this->author = $post->author;

$this->body = $post->body;

$this->date = $post->date->format('Y-m-d');

return $clone;

}

}

```

### High Order Tests

```php
$post = Post::factory()->create();

$updatedPost = $post->refresh();

expect($updatedPost)
	->title->toBe($requestData['title'])
	->author->toBe($requestData['author'])
	->body->toBe($requestData['body'])
	->date->toBe($reuqestData['date']);

expect($post->postLikes)
	->toHaveCount(2)
	->sequence(
		fn($like) => $like->like_uuid->toBe('a')->another_attribute->toBe('value'),
		fn($like) => $like->like_uuid->toBe('b')
	);
```

### API Test

jsonの検証方法は２個ある。

```php
$posts = BlogPost::factory()->count(5)->create();

getJson(action([BlogPostController::class, 'index']))
	->assertSuccessful()
	->assertJson(function(AssertableJson $json) use $posts {
		$json
			->where('id', $posts[0]->id)
			->where('slug', $posts[0]->slug)
			->where('date', $posts[0]->date->toDateTimeLocalString(). '.000000Z')
			->whereAllType([
				'id' => 'integer',
				'slug' => 'string',
				'date' => 'string'
			])
			->etc();
			
	});

getJson(action([BlogPostController::class, 'show'], $post))  
 ->assertSuccessful()  
 ->assertJsonPath('data.title', $post->title);

```

### beforeEach
Custom FactoryはbeforeEachでnewしておくと使いやすい

```php
beforeEach(function () {  
 $this->post = BlogPost::factory()->create();  
    $this->requestData = BlogPostRequestDataFactory::new();  
});
```