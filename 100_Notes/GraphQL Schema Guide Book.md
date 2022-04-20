### 命名規則

#### Query

- リソース名（名詞）とその複数形
- `repository`, `repositories`

#### Mutation

- 動詞＋名詞の形 `addStar`, `createIssue`
	- Mutationの名前+Input
	- Mutationの名前+Payload

### Global Object Identification

- 全データをIDにより一意に特定できるようにする仕様
- users tableのidを一意にしたければ `User:1`など

```graphql
type Query {
	node(id: ID!): Node
}

interface Node {
	id: ID!
}

type User implements Node {
	id: ID!
	name: String
}
```

- Nodeに型指定をするとデータが取得する

```graphql
{
  node(id: "MDEyOk9yZ2FuaXphdGlvbjEzNDIwMDQ=") {
    __typename
    id
    ... on Organization {
      databaseId
      login
    }
  }
}
```

- `nodes(ids: [ID!]!): [Node]!`もついでに定義するとよい

### Cursor Connections

- カーソルを使ってリストを順にたどるための仕様
	- `Repository`のリストを返す場合、`[Repository]`を返す代わりに`RepositoryConnection`を返す

1. リスト型のフィールドの代わりに、Connectionサフィックスの型を作る
2. first: Int!とafter: Stringを引数に設ける
3. Connectionはcursorを持てるEdgeとページング情報をもつPageInfoをもつ

```graphql

type Query {
	repositories(
		first: Int
		after: String
		
		last: Int
		before: String
	): RepositoryConnection
}

type RepositoryConnection {
	pageInfo: PageInfo
	edges: [RepositoryEdge]
}

type PageInfo {
	hasPreviousPage: Boolean!
	startCursol: String
	hasNextPage: Boolean!
	endCursol: String
}

type RepositoryEdge {
	cursor: String!
	node: Repository
}

type Repository {
	id: ID!
	name: String!
}

```

### Queryでinputはアンチパターン？

- フロント側的には`input`でまとめなくても`variables`でまとまる
- 引数を明示的に指定するのは2 ~ 3がほとんどであまり多くならない
- デフォルト値の指定が活用できずつらい

