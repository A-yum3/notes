## Schema

### Type

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  created_at: String!
  updated_at: String
}
```

#### Object Type

Eloquentモデルと密接に関連していて、一意の名前を持ち、一連のフィールドを含んでいる必要がある。

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  created_at: String!
  updated_at: String
}

type Query {
  users: [User!]!
  user(id: ID!): User
}
```

#### Scalar

- [Lighthouseデフォルトのスカラー型](https://lighthouse-php.com/5/api-reference/scalars.html)
- `php artisan lighthouse:scalar <Scalar name>`で独自のスカラー型を定義することができる
- サードパーティによる[mll-lab/graphql-php-scalars](https://github.com/mll-lab/graphql-php-scalars)スカラーが型も使用できる
- [独自のスカラー型を実装する](https://webonyx.github.io/graphql-php/type-definitions/scalars/)

#### Enum

`@enum`ディレクティブを使用して値を定義できる

```graphql
enum EmploymentStatus {
  INTERN @enum(value: 0)
  EMPLOYEE @enum(value: 1)
  TERMINATED @enum(value: 2)
}
```

https://lighthouse-php.com/5/the-basics/types.html#enum

### Root Type

#### Query

```graphql
type Query {
  me: User
  users: [User!]!
  userById(id: ID): User
}
```

#### Mutation

```graphql
type Mutation {
  createUser(name: String!, email: String!, password: String!): User
  updateUser(id: ID, email: String, password: String): User
  deleteUser(id: ID): User
}
```

#### Subscription

```graphql
type Subscription {
  newUser: User
}
```

### Fields

- デフォルトでは、Lighthouseは `App\GraphQL\Queries`また`App\GraphQL\Mutations`の名前空間下でフィールドの大文字の名前を持つクラスを検索し、`__invoke`を使い、 [通常のリゾルバ引数](https://lighthouse-php.com/5/api-reference/resolvers.html#resolver-function-signature)を使用して呼び出す。
- `php artisan lighthouse:query <QueryName>` で生成できる
- `$args`はクエリに渡される引数の連想配列

### ディレクティブ

 - `@`で始まり、一意の名前が続く
 - [Directives API](https://lighthouse-php.com/5/api-reference/directives.html)

## Eloquent
- READ
	- `@all`でモデルの名前が解決しているフィールドの戻りタイプと同じであると想定し、Eloquentで自動解決する
	- `@paginate`でページネーション
	- `@eq`で追加制約をクエリに追加できる
	- `@orderBy`でソート
- CREATE
	- `@create`でデータ作成
	- `@update`でモデル更新
	- `@upsert`
	- `@delete`

## スキーマ構成

- `schema.graphql`からスキーマを読み取る
- インポートにワイルドカードも使える
- 

```graphql
type Query {
  user: User
}

#import user.graphql
```