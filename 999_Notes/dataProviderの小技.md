#php/laravel/test 

dataProviderはGeneratorであれば作用する。

つまり、引数をClosureにして、yieldで\[関数\]のような形式にすれば、そのまま関数を渡すことができる。


```php
/**

* @test

* @dataProvider requests

*/

public function admin_are_allowed(Closure $sendRequest)

{

$this->login(User::factory()->admin()->create());

$post = BlogPost::factory()->create();

/** @var \Illuminate\Testing\TestResponse $response */

$response = $sendRequest->call($this, $post);

$this->assertTrue(in_array($response->status(), [302, 200]));

}

public function requests(): Generator

{

yield [fn (BlogPost $post) => $this->get(action([BlogPostAdminController::class, 'index']))];

yield [fn (BlogPost $post) => $this->get(action([BlogPostAdminController::class, 'create']))];

yield [fn (BlogPost $post) => $this->post(action([BlogPostAdminController::class, 'store']))];

yield [fn (BlogPost $post) => $this->get(action([BlogPostAdminController::class, 'edit'], $post->slug))];

yield [fn (BlogPost $post) => $this->post(action([BlogPostAdminController::class, 'update'], $post->slug))];

yield [fn (BlogPost $post) => $this->post(action([BlogPostAdminController::class, 'publish'], $post->slug))];

yield [fn (BlogPost $post) => $this->post(action(UpdatePostSlugController::class, $post->slug))];

yield [fn (BlogPost $post) => $this->post(action(UpdatePostSlugController::class, $post->slug))];

yield [fn (BlogPost $post) => $this->post(action(DeletePostController::class, $post->slug))];

}
```