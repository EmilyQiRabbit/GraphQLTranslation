> * 原文地址：[Core Concepts](https://www.howtographql.com/basics/2-core-concepts/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 核心概念

在这一章，你将会学到 GraphQL 的基础语言结构。包括类型定义，发送请求和修改的语法。[戳这里](https://www.howtographql.com/basics/2-core-concepts/) 为你准备了可以运行代码的沙箱，它基于 [graphql-up](https://github.com/graphcool/graphql-up)，可以用来测试代码。

## The Schema Definition Language (SDL)：Schema 定义语言

GraphQL 有它自己的类型系统，用来定义 API 的 schema。用来写 schema 的语法就叫 Schema Definition Language (SDL)。

下面是一个简单的栗子，用 SDL 定义一个 Person 类：

```JavaScript
type Person {
  name: String!
  age: Int!
}
```

这个类型有两个字段（field）：name 和 age，分别是 String 类型和 Int 类型。类型后的 ! 符号表示这个字段的值是必须的。

类型之间的关系也可以用 SDL 来定义。下面这个例子描述了博客 Post 和人 Person 之间的关系：

```JavaScript
type Post {
  title: String!
  author: Person!
}
```

反过来，Person 类型也需要和 Post 关联：

```JavaScript
type Person {
  name: String!
  age: Int!
  posts: [Post!]!
}
```

我们现在建立了一个 Person - Post 的一对多关系，Person 的 posts 字段是表示个人所有博客的数组。

## 使用 Queries 获取数据

如果使用 REST APIs 来获取数据，那么数据是从特定的接口加载的。每个接口都明确定义了返回的数据格式。这就意味着，客户端对数据的需求都有效的编码在了它链接的 URL 中。

GraphQL 的数据请求就大不相同了。它只暴露一个接口，而不是多个数据结构固定的接口。这个接口返回的数据结构也不固定。这就相当灵活了，并且允许客户端决定它所需要的数据格式。

这就意味着，客户端需要向服务器发送更多的信息来描述它需要的数据 - 这也就是 query。

### 最基本的 Queries

看一个客户端发送给服务器的 query 的例子：

```JavaScript
{
  allPersons {
    name
  }
}
```

query 中的 allPersons 字段叫做根字段（root field）。root field 字段包含的值，就是这个 query 的有效负荷（payload）。这个 query 规定了的 payload 中的字段只有 name。

这个 query 将会返回数据库中所有的 person 的一个列表，这就是一个应答的例子：

```JavaScript
{
  "allPersons": [
    { "name": "Johnny" },
    { "name": "Sarah" },
    { "name": "Alice" }
  ]
}
```

我们注意到，服务器只返回了每个 person 的 name 字段，但是 age 并没有被返回。这就是因为 query 中只定义了 name 字段。

如果客户端也需要 age 信息，那么只需要稍稍修改 query，在 payload 中增加一个新 age 字段：

```JavaScript
{
  allPersons {
    name
    age
  }
}
```

GraphQL 的一个主要优点就是：它原生的允许 query 嵌套信息。例如，如果你想要加载一个 person 的所有 post 信息，那么你只需要这样来构建你的 query 来发起请求：

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

### 带参数的 Queries

在 GraphQL，每一个 field 字段都可以带零到多个参数。例如，字段 allPersons 可以有一个 last 参数来定义返回的 person 的个数：

```JavaScript
{
  allPersons(last: 2) {
    name
  }
}
```

## 使用 mutation 来写入数据

下面我们来学习如何修改服务端存储的数据。GraphQL 使用所谓的 mutation 来完成这项任务，通常情况下，有三种类型的 mutation：

* 新建
* 更新
* 删除

Mutations 和 queries 有同样的语法结构，但是需要以 mutation 关键字开头。下面是一个新建一个 Person 的方法：

```JavaScript
mutation {
  createPerson(name: "Bob", age: 36) {
    name
    age
  }
}
```

这和我们刚才写的 query 很相似，mutation 也有一个 root 字段 - 在这个例子中就是 createPerson。我们已经知道了字段可以带参数，这个例子中，createPerson 字段有两个参数：name 和 age。

就像 query 一样，我们也能够为 mutation 定义 payload，在 payload 中可以请求新创建的的对象的某些属性信息。在这个例子中，我们请求了 name 和 age。这个例子中获取这两个信息其实并不对我们有特别的帮助，因为我们已经知道了它们的值。但是，发送 mutation 的时候同时允许 query 这个功能其实非常强大，它允许你在一次请求中同时也从服务器获取新的信息。

对于上面的 mutation，服务端的回应是：

```JavaScript
"createPerson": {
  "name": "Bob",
  "age": "36",
}
```

当新的对象被创建的时候，服务器将会自动的创建一个唯一的 ID。所以我们可以为之前的 Person type 添加一个 id 字段：

```JavaScript
type Person {
  id: ID!
  name: String!
  age: Int!
}
```

现在，如果创建了一个新的 Person，就可以直接在 mutation 的 payload 中请求 id，因为这个信息是之前客户端并不存在的。

```JavaScript
mutation {
  createPerson(name: "Alice", age: 36) {
    id
  }
}
```

## 使用 Subscriptions 实时更新

另外一个对现在很多应用都很重要的功能，就是和服务器的实时连接，以方便及时的获取重要消息。对此 GraphQL 提供了 subscriptions 这个概念。

当客户端订阅了某个事件，它就将会初始化一个链接并且和服务器保持这个链接。这样，管什么时候这个特定的事件被触发了，服务器都会推送给客户端相应的信息。不像 query 和 mutation 那样的请求-应答循环，subscriptions 代表了一个由服务端向客户端的数据流。

subscriptions 的语法和 query、mutation 也很像，下面是一个例子，订阅了 Person type 的事件：

```JavaScript
subscription {
  newPerson {
    name
    age
  }
}
```

客户端向服务器发送了这个 subscription 之后，它们之间的链接就建立了。然后，每当 mutation 被执行并且创建了一个新的 Person，服务器就会给客户端发送这个人的信息：

```JavaScript
{
  "newPerson": {
    "name": "Jane",
    "age": 23
  }
}
```

## 定义 Schema

现在我们已经知道了 queries，mutations 和 subscriptions 的基本格式是什么样了，我们总结一下然后学习如何写 schema，正是它允许你执行刚才所有的这些例子。

schema 是 GraphQL API 中最重要的概念，它定义了 API 能做什么，也定义了客户端能够如何请求数据。它可以被视作服务器和客户端之间的协议。

通常情况下，schema 就是 GraphQL types 的集合。但是，当 API 写 schema 的时候，首先需要一些特定的 root type：

```JavaScript
type Query { ... }
type Mutation { ... }
type Subscription { ... }
```

Query，Mutation 和 Subscription 类型就是客户端发送请求的入口。为了支持刚才的 allPersons query，我们需要在 Query type 中这样写：

```JavaScript
type Query {
  allPersons: [Person!]!
}
```

allPersons 就是这个 API 的 root 字段。刚才我们还为 allPersons 添加了一个 last 参数，那么我们应该这样写：

```JavaScript
type Query {
  allPersons(last: Int): [Person!]!
}
```

相似的，对于 createPerson mutation，我们则需要在 Mutation type 下添加：

```JavaScript
type Mutation {
  createPerson(name: String!, age: Int!): Person!
}
```

注意到，这个 root 字段有两个参数：Person 的 name 和 age。

最后，为了支持 subscriptions，我们也需要添加 newPerson 的 root 字段：

```JavaScript
type Subscription {
  newPerson: Person!
}
```

总结一下这一章，然后把所有代码放在一起，所有用来支持 Query，Mutation 和 Subscription 的 schema 就是：

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

🎉

[self Proofreading +1]