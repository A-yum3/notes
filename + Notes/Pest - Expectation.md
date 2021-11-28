#php/pest 

https://pestphp.com/docs/expectations
Jestを使った人なら馴染みがあると思います。

`expect($value)`に検査したい$valueをセットし、あとはチェーンで検証したいことを連ねます。

| PHPUnit                   | Pest                                        | 備考                                            |
| ------------------------- | ------------------------------------------- | ----------------------------------------------- |
| assertSame                | toBe($expected)                             |                                                 |
| assertEmpty               | toBeEmpty()                                 |                                                 |
| assertTrue                | toBeTrue()                                  |                                                 |
| -                         | toBeTruthy()                                |                                                 |
| assertFalse               | toBeFalse()                                 |                                                 |
| -                         | toBeFalsy()                                 |                                                 |
| assertGreaterThan         | toBeGreaterThan($expected)                  |                                                 |
| assertGreaterThanOrEqual  | toBeGreaterThanOrEqual($expected)           |                                                 |
| assertLessThan            | toBeLessThan($expected)                     |                                                 |
| assertLessThanOrEqual     | toBeLessThanOrEqual($expected)              |                                                 |
| assertContains            | toContain($needles)                         | 複数                                            |
|                           | toHaveCount(int $count)                     |                                                 |
| assertObjectHasAttribute  | toHaveProperty(string $name, $value = null) |                                                 |
| -                         | toHaveProperties(iterable $name)            |                                                 |
| -                         | toMatchArray($array)                        | 部分一致                                        |
| -                         | toMatchObject($object)                      | 部分一致                                        |
| assertEqual               | toEqual($expected)                          |                                                 |
| -                         | toEqualCanonicalizing($expected)            | 正規化(ソートなど)される                        |
| -                         | toEqualWithDelta($expected, float $delta)   | 差の絶対値がdeltaより小さい時pass               |
| assertContains            | toBeIn($value)                              | $valueに含まれていればpass                      |
| -                         | toBeInfinite()                              |                                                 |
| assertInstanceOf          | toBeInstanceOf($class)                      |                                                 |
| assertIsArray             | toBeArray()                                 |                                                 |
| assertIsBool              | toBeBool()                                  |                                                 |
| assertIsCallable          | toBeCallable()                              |                                                 |
| assertIsFloat             | toBeFloat()                                 |                                                 |
| assertIsInt               | toBeInt()                                   |                                                 |
| assertIsIterable          | toBeIterable()                              |                                                 |
| assertIsNumeric           | toBeNumeric()                               |                                                 |
| assertIsObject            | toBeObject()                                |                                                 |
| assertIsResource          | toBeResource                                |                                                 |
| assertIsScalar            | toBeScalar()                                |                                                 |
| assertIsString            | toBeString()                                |                                                 |
| -                         | toBeJson()                                  |                                                 |
| assertNan                 | toBeNan()                                   |                                                 |
| assertNull                | toBeNull()                                  |                                                 |
| assertArrayHasKey         | toHaveKey(string $key)                      | 第2引数を渡せば値も検証可能で、ドット表記も対応 |
| -                         | toHaveKeys(array $keys)                     |                                                 |
| -                         | toHaveLength(int $number)                   | 文字列・iterableの数                            |
| assertDirectoryExists     | toBeDirectory()                             |                                                 |
| assertDirectoryIsReadable | toBeReadableDirectory                       |                                                 |
| assertDirectoryIsWritable | toBeWritableDirectory                       |                                                 |
| assertStringStartWith     | toStartWith(string $expected)               |                                                 |
| -                         | toThrow()                                   | notも使える                                     |
| assertRegExp              | toMatch(string $expression)                 |                                                 |
| -                         | toEndWith(string $expected)                 |                                                 |
| -                         | toMatchConstraint(Constraint $constraint)   |                                                 |
| -                         | and($value)                                 | 2個目以降のexpect                               |
| -                         | dd()                                        |                                                 |
| -                         | each()                                      | iterableの各要素に                              |
| -                         | json()                                      | json文字列をjson化してテスト可能にする          |
| -                         | match()                                     |                                                 |
| -                         | not()                                       | チェーンの前において反転させる                  |
| -                         | sequence()                                  |                                                 |
| -                         | when()                                      |                                                 |
| -                         | unless()                                    |                                                 |

### カスタム Expectations
Expectationsを組み合わせて作るオリジナルExpectations

`expect()->extend()`で拡張できる。