> * 原文地址：[Core Concepts](https://www.howtographql.com/basics/2-core-concepts/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 核心概念

在这一章，我们将会一起学习 GraphQL 的一些基础语言结构。包括定义类型的语法，以及发送查询和修改请求的语法。我们也为你准备了一个沙箱环境，它基于 [graphql-up](https://www.npmjs.com/package/graphql-up)，可以用来测试你学习到的代码。

## 模式定义语言：即 Schema Definition Language，SDL

GraphQL 有一套自己的类型系统，可以用来定义 API 的模式（即 schema）。用来写框架的语法就称为模式定义语言（即 Schema Definition Language，SDL）。

下面是一个简单的例子，我们可以用 SDL 定义一个 `Person` 类型：

```JavaScript
type Person {
  name: String!
  age: Int!
}
```

这个类型有两个字段：`name` 和 `age`，分别是 `String` 类型和 `Int` 类型。类型后的 `!` 符号表示这个字段的值是必需的。

类型间关系也可以用 SDL 来定义。下面这个例子描述了博客（`Post`）和人（`Person`）之间的关系：

```JavaScript
type Post {
  title: String!
  author: Person!
}
```

反过来说，`Person` 类型也需要和 `Post` 关联：

```JavaScript
type Person {
  name: String!
  age: Int!
  posts: [Post!]!
}
```

此时我们就建立了一个 `Person` - `Post` 的一对多关系，`Person` 的 `posts` 字段是表示当前用户的所有博客的数组。

## 使用 query 获取数据

使用 REST API 获取数据的时候，数据需要从特定的某几个接口加载。每个接口都明确定义了返回值的格式。这就意味着客户端对数据的需求都高效的编码在了它所请求的 URL 中。

而 GraphQL 则大不相同。GraphQL API 通常只暴露一个接口，而不是多个返回结构固定的数据的接口。这个接口返回的数据结构并不固定。这非常灵活，并且允许客户端决定它所需要的数据格式。

这就意味着，客户端需要向服务端发送更多的信息来描述它需求的数据类型 - 这些信息称为 query。

### 基本的 query

看一个客户端发送给服务端的 query 的例子：

```JavaScript
{
  allPersons {
    name
  }
}
```

> 译者注：沙箱演示需要在原文中查看。

query 中的 `allPersons` 字段称为根字段（root field）。根字段的值，就是这个 query 的数据负载（payload）。这个 query 的数据负载中指定的字段只有 `name`。

这个 query 将会返回一个包含了数据库中所有 person 的一个列表，例如：

```JavaScript
{
  "allPersons": [
    { "name": "Johnny" },
    { "name": "Sarah" },
    { "name": "Alice" }
  ]
}
```

我们注意到，服务端接口只返回了每个 person 的 name 字段，但是 age 并没有被返回。这是因为 query 中只指定了 name 字段。

如果客户端也需要用户的 age 信息，那么只需要稍稍修改 query，在数据负载中增加一个新 age 字段：

```JavaScript
{
  allPersons {
    name
    age
  }
}
```

GraphQL 的一个最大的优点就是：它原生支持查询的信息嵌套。例如，如果你想要加载一个用户的所有博客信息，那么你只需要根据类型结构来像这样请求数据：

```JavaScript
{
  allPersons {
    name
    age
    posts {
      title
    }
  }
}
```

### 带参数的 query

在 GraphQL 的 query 中，每个字段都可以携带零到多个参数，但要求这些参数是在模式（schema）中定义过或者支持的。例如，根字段 `allPersons` 可以携带一个 `last` 参数，定义返回的用户个数。相应的 query 就可以这样写：

```JavaScript
{
  allPersons(last: 2) {
    name
  }
}
```

## 使用 mutation 写入数据

在知道了如何从服务端请求数据后，大部分应用还需要修改保存在后端的数据。GraphQL 使用所谓的 mutation 来完成这项任务，通常情况下，有三种类型的 mutation：

* 新建数据
* 更新数据
* 删除数据

mutation 和 query 有同样的语法结构，但是需要以 mutation 关键字开头。下面是新建 `Person` 的方法：

```JavaScript
mutation {
  createPerson(name: "Bob", age: 36) {
    name
    age
  }
}
```

这和我们刚才写的 query 很相似，mutation 也有一个根字段 - 在这个例子中就是 `createPerson`。我们已经知道了字段可以携带参数，这个例子中，`createPerson` 字段有两个参数：`name` 和 `age`，用来定义这个新用户的名字和年龄。

和 query 一样，我们也能够为 mutation 定义数据负载，数据负载可以用来请求该新创建的对象的某些属性信息。在这个例子中，我们请求了 name 和 age —— 但其实在这个例子中，获取这两个字段也没什么用，因为我们在发送 mutation 的时候就已经知道了它们的值。但是，发送 mutation 的时候同时允许请求其他信息这个功能非常强大，因为它允许你在一次请求中同时从服务端取回新的信息。

对于上面的 mutation，服务端的回应是：

```JavaScript
"createPerson": {
  "name": "Bob",
  "age": "36",
}
```

当新的对象被创建的时候，服务器将会自动的为它创建一个唯一的 ID。所以我们可以为之前的 `Person` 类型添加一个 `id` 字段：

```JavaScript
type Person {
  id: ID!
  name: String!
  age: Int!
}
```

现在，当一个新的 `Person` 被创建之后，我们就可以直接在 mutation 的数据负载中请求它的 `id`，因为这个信息之前在客户端并不存在。

```JavaScript
mutation {
  createPerson(name: "Alice", age: 36) {
    id
  }
}
```

## 使用 subscription 进行实时更新

另外一个大多数应用中都很重要的功能就是和服务端进行实时连接，这样用户就可以及时获取到重要消息。对此，GraphQL 提供了 subscription 这个概念。

当某个客户端订阅了某个事件，它就会初始化一个和服务端的连接，并一直保持这个连接不中断。这样，不管这个特定的事件什么时候被触发，服务端都会推送相应的信息给这个客户端。subscription 不像 query 和 mutation 那样需要遵循一个“请求 - 应答的循环”，它表示一个从服务端到客户端的数据流。

subscription 的语法和 query、mutation 也很像，下面是一个例子，我们订阅了 Person 类型上的事件：

```JavaScript
subscription {
  newPerson {
    name
    age
  }
}
```

客户端向服务端发送了这个 subscription 之后，它们之间的连接就建立起来。然后每当一个 mutation 被执行并且创建了一个新的 `Person`，服务端就会向客户端发送这个新创建的数据：

```JavaScript
{
  "newPerson": {
    "name": "Jane",
    "age": 23
  }
}
```

## 定义一个模式（schema）

现在我们已经了解了 query，mutation 和 subscription 的基础知识。我们来总结一下，然后学习如何写模式。正是由于有了模式，服务端才能允许你执行刚才所有的这些代码。

模式是使用 GraphQL API 时需要懂得的最重要的概念，它定义了 API 功能，也定义了客户端应该如何请求数据。它通常被视作服务端和客户端之间的协议。

通常情况下，模式就是 GraphQL 类型的集合。但是，当为 API 写模式文件的时候，首先需要一些特定的根类型（root type）：

```JavaScript
type Query { ... }
type Mutation { ... }
type Subscription { ... }
```

`Query`，`Mutation` 和 `Subscription` 类型就是客户端发送的请求的入口。为了支持刚才的 `allPersons` query，我们需要在 `Query` 这个根类型中写这样的代码：

```JavaScript
type Query {
  allPersons: [Person!]!
}
```

`allPersons` 就是这个 API 的根字段。想到刚才我们还为 `allPersons` 字段添加了一个 `last` 参数，那么我们应该这样写：

```JavaScript
type Query {
  allPersons(last: Int): [Person!]!
}
```

类似的，对于 `createPerson` 这个 mutation，我们则需要在 `Mutation` 类型下添加：

```JavaScript
type Mutation {
  createPerson(name: String!, age: Int!): Person!
}
```

注意到，这个根字段有两个参数：`Person` 的 `name` 和 `age`。

最后，为了支持 subscription，我们也需要添加 `newPerson` 这个根字段：

```JavaScript
type Subscription {
  newPerson: Person!
}
```

最后，我们把所有代码都放在一起，本章所有用来支持 query 和 mutation 的**完整的**模式就是：

```JavaScript
type Query {
  allPersons(last: Int): [Person!]!
}

type Mutation {
  createPerson(name: String!, age: Int!): Person!
}

type Subscription {
  newPerson: Person!
}

type Person {
  name: String!
  age: Int!
  posts: [Post!]!
}

type Post {
  title: String!
  author: Person!
}
```
## 扩展阅读：

[GraphQL Server Basics (Part I): GraphQL Schemas, TypeDefs & Resolvers Explained](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e)

[GraphQL Server Basics (Part II): The Network Layer](https://www.prisma.io/blog/graphql-server-basics-the-network-layer-51d97d21861)

[GraphQL Server Basics (Part III): Demystifying the info argument in GraphQL resolvers](https://www.prisma.io/blog/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a)
