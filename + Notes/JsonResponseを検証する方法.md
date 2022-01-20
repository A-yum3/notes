#php/laravel/test 

- assertJsonを使用し、中でAssertableJsonで検証したい部分を指定する

```php
$this
    ->get(action(PostIndexController::class))
    ->assertSuccessful()
    ->assertJson(fn (AssertableJson $json) =>
        $json
            ->has('data', 2)
            ->has(
                'data.0',
                fn (AssertableJson $json) =>
                    $json
                        ->has('id')
                        ->has('date')
                        ->has('slug')
                        ->etc()
            )
    );
```