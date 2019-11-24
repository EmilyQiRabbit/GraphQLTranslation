> * 原文地址：[Server](https://www.howtographql.com/advanced/1-server/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 更多 GraphQL 概念

## 使用 fragment 增强可复用性

fragment 是可以帮助你优化 GraphQL 代码结构和可复用性的一个很方便的特性。fragment 是一个特殊类型字段的集合。

假设我们有如下这样的类型：

```graphql
type User {
  name: String!
  age: Int!
  email: String!
  street: String!
  zipcode: String!
  city: String!
}
```

这里，我们在一个 fragment 中包含了代表用户所有地理位置相关的信息：

```graphql
fragment addressDetails on User {
  name
  street
  zipcode
  city
}
```

现在，当需要写一个用来请求用户地址信息的 query 的时候，我们就可以使用如下这样的语法，它引用了之前的 fragment，节约了再次拼写那 4 个字段的时间：

```graphql
{
  allUsers {
    ... addressDetails
  }
}
```

这两种写法是等价的：

```graphql
{
  allUsers {
    name
    street
    zipcode
    city
  }
}
```

## 参数化字段

在 GraphQL 的类型定义中，每个字段都可以带 0 到多个参数。和其他强类型编程语法中，传递入函数的参数很类似，每个参数都需要定义参数名和参数类型。在 GraphQL 中，也可以指定参数的默认值。

例如，让我们一起考虑开头那个 schema 的这一部分：

```graphql
type Query {
  allUsers: [User!]!
}

type User {
  name: String!
  age: Int!
}
```

我们可以为 allUsers 字段增加一个参数，这个参数让我们可以以年龄为条件过滤用户。我们也可以规定一个默认值，这样在默认情况下，所有的用户都会被返回：

```graphql
type Query {
  allUsers(olderThan: Int = -1): [User!]!
}
```

这个 olderThan 参数可以以这样的格式传递给 query：

```graphql
{
  allUsers(olderThan: 30) {
    name
    age
  }
}
```

## 使用别名命名 query 请求的结果

GraphQL 的最强大的地方之一在于，你能够在一次请求中发送多个 query。但是，既然服务端返回的数据和发送请求的字段结构是保持一致的，那么当你发送多个请求相同字段的 query 的时候，你也许就可能遇到命名相关的问题：

```graphql
{
  User(id: "1") {
    name
  }
  User(id: "2") {
    name
  }
}
```

事实上，由于相同的字段使用了不同的参数，这会在 GraphQL 服务中造成错误。想要这样发送多个 query 的唯一方法就是使用别名，例如，为 query 的结果指定不同的名字：

```graphql
{
  first: User(id: "1") {
    name
  }
  second: User(id: "2") {
    name
  }
}
```

结果就是，在服务端返回的结果中，将会根据指定的别名为每个 User 对象命名：

```graphql
{
  "first": {
    "name": "Alice"
  },
  "second": {
    "name": "Sarah"
  }
}
```

## 高级 SDL（即 Schema Definition Language）

SDL 提供了很多语言特性，有很多我们在前文中都并没有提到过。接下来，我们将会一起通过实例讨论它们：

### object 和 scalar 类型

在 GraphQL 中，有两种不同的类型：

* scalar 类型代表了具体的数据单元。GraphQL 规定有五种预定义的 scalar，即 String、Int、Float、Boolean 和 ID。

* object 类型可以有多个字段，用来描述该类型的属性，同时它们也是可组合的。我们在前面的章节中看到的 User 和 Post 类型就是 object 类型的例子。

在每个 GraphQL 服务的 schema 中都可以定义你自己的 scalar 和 object 类型。一个经常被引用的用户自定义 scalar 类型就是 Date 类型，它的实现需要定义如何验证、序列化和反序列化该类型。

### enums 类型

GraphQL 允许定义枚举类型，简写为 enums，这种语言特性可以描述有一系列固定值的类型。由此，我们可以定义一个 Weekday 类型，来代表一周内的每一天：

```graphql
enum Weekday {
  MONDAY
  TUESDAY
  WEDNESDAY
  THURSDAY
  FRIDAY
  SATURDAY
  SUNDAY
}
```

注意，从技术上来说，enums 属于特殊类型的 scalar 类型。

### interface 类型

使用 interface，我们可以以很抽象的方式描述类型。它让你能定义一系列任意类型字段的集合，所有实现了这个接口的类型，都必须要包含这些字段。在很多 GraphQL schema 中，所有类型都必须有一个 id 字段。如果使用 interface，这个需求就可以通过定义一个包含 id 字段的 interface 来描述，并保证所有用户定义的类型都要实现这个接口：

```graphql
interface Node {
  id: ID!
}

type User implements Node {
  id: ID!
  name: String!
  age: Int!
}
```

### union 类型

union 类型可以用来描述，一个类型属于其他类型集合中的一个。最好的理解方式还是通过举例子。我们假设有如下类型：

```graphql
type Adult {
  name: String!
  work: String!
}

type Child {
  name: String!
  school: String!
}
```

现在，我们定义一个 Person 类型，它是 Adult 和 Child 的 union 类型：

```graphql
union Person = Adult | Child
```

这就造成了另外一个问题：我们在一个 GraphQL query 中，请求一个 Child 类型的信息，但我们目前只有 Person 类型可以使用，我们该如何知道我们真的可以访问属于 Child 的字段呢？

问题的答案，我们称之为条件 fragment：

```graphql
{
  allPersons {
    name # works for `Adult` and `Child`
    ... on Child {
      school
    }
    ... on Adult {
       work
    }
  }
}
```
