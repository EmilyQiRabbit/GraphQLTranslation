> * 原文地址：[Realtime GraphQL Subscriptions](https://www.howtographql.com/graphql-js/7-subscriptions/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 实时 GraphQL 订阅

在这一章节种，你将会学习如何通过实现 GraphQL subscriptions 为应用加入实时更新的功能。目标是实现两个 GraphQL 服务暴露出来的的订阅功能：

* 当一个新的 Link 被创建的时候为订阅用户推送实时更新。

* 当一个已存在的 Link 被点赞的时候为订阅用户推送实时更新。

## 什么是 GraphQL subscriptions？

订阅（subscriptions）是 GraphQL 的一个功能，它允许服务器在发生某些特定事件的时候，向客户端发送数据。订阅功能常常使用 WebSockets 实现。本篇配置中，服务器和参与订阅的客户端维持了一个持续的连接。它打破了“请求-回复”这样的之前所有 API 交互的模式。

## 使用 Prisma 的订阅

幸运的是，Prisma 提供了开箱即用的订阅功能。事实上，如果你看看 src/generated/prisma.graphql 文件中的 Prisma schema，你就会注意到 Subscription 功能已经存在于文件中了：

```js
type Subscription {
  link(where: LinkSubscriptionWhereInput): LinkSubscriptionPayload
  user(where: UserSubscriptionWhereInput): UserSubscriptionPayload
}
```

这几个 Subscription 将会在下列事件发生的时候被触发：

* 建立了一个新的节点

* 更新了节点

* 节点删除

注意，你可以约束触发订阅函数的事件。例如，如果你只想订阅某个 Link 的更新或者某个特定用户的删除。你可以功过提供给 subscription 请求 where 参数来实现。

GraphQL subscription 和 mutation(修改) / query(请求) 遵循同样的语法结构，所以你可以像下面这样订阅 Link 更新事件：

```js
subscription {
  link(where: {
    mutation_in: [UPDATED]
  }) {
    node {
      id
      url
      description
    }
  }
}
```

这个 subscription 将会在某个 Link 被更新的时候触发，服务端就会发送被更新的 Link 的 url 和 description。

我们也来快速学习一下 src/generated/prisma.graphql 中的类型 LinkSubscriptionPayload：

```js
type LinkSubscriptionPayload {
  mutation: MutationType!
  node: Link
  updatedFields: [String!]
  previousValues: LinkPreviousValues
}
```

下面是每个字段的含义：

### mutation: MutationType!

MutationType 是一个有三个值的枚举：

```js
enum MutationType {
  CREATED
  UPDATED
  DELETED
}
```

这个字段代表了 mutation 的类型

### node: Link

这个字段代表被创建、更新或者删除的 Link，并且允许追溯它的更多信息。

注意：DELETE mutation 发生的时候，node 就一定是 null。如果你想要知道被删除的 Link 的更多信息，你可以使用稍后将会介绍的 previousValues 字段。

### updatedFields: [String!]

在 update mutation 中你最可能感兴趣的信息就是哪个字段被更新了。updatedFields 这个字段就是用来提供这个信息的。

假设一个客户端订阅了如下 subscription query：

```js
subscription {
  link {
    updatedFields
  }
}
```

现在，假设服务器收到了如下的 mutation：

```js
mutation {
  updateLink(
    where: {
      id: "..."
    }
    data: {
      description: "An even greater website"
    }
  )
}
```

那么订阅了的客户端将会受到如下的信息：

```js
{
  "data": {
    "link": {
      "updatedFields": ["description"]
    }
  }
}
```

因为刚才发送的 mutation 仅仅修改了 description 字段。

### previousValues: LinkPreviousValues

LinkPreviousValues 类型和 Link 类型非常相似：

```js
type LinkPreviousValues {
  id: ID!
  description: String!
  url: String!
}
```

它基本就是 Link 类型的镜像，是一个辅助类型。

previousValues 仅仅在 UPDATE 和 DELETE mutation 的时候使用。在 CREATION mutation 的时候，它总是空：null。

**现在，把所有这些总结一下。**

假设前文的 updateLink mutation 不变，但是 subscription 包含上面我们讨论过的所有字段：

```js
subscription {
  link {
    mutation
    updatedFields
    node {
      url
      description
    }
    previousValues {
      url
      description
    }
  }
}
```

于是下面这就是，当 mutation 发生后，服务端推送给客户端的信息：

```js
{
  "data": {
    "link": {
      "mutation": "UPDATED",
      "updatedFields": ["description"],
      "node": {
        "url": "www.example.org",
        "description": "An even greater website"
      },
      "previousValues": {
        "url": "www.example.org",
        "description": "A great website"
      }
    }
  }
}
```

注意：我们假设在修改之前，Link 的值为：

```js
url: www.example.org
description: A great website
```

## 订阅新建 Link 元素

停止纸上谈兵，我们来写一写代码：实现一个允许客户端订阅新 Link 被创建信息的订阅函数。

就和 query 和 mutation 一样，第一步是去扩充 GraphQL schema 的定义。

打开 schema 并添加 Subscription 类型：

```js
type Subscription {
  newLink: LinkSubscriptionPayload
}
```

由于引用了 Prisma schema 的 LinkSubscriptionPayload 类型，你也需要调整下文件头部的 import 语句：

```js
# import Link, LinkSubscriptionPayload from "./generated/prisma.graphql"
```

下面，开始实现 newLink 字段的 resolver 函数。subscription 的 resolver 函数和 query / mutation 的稍微有一些区别：

1. 它不会直接返回任何数据，它返回一个异步迭代器 (AsyncIterator)，这个迭代器将会被 GraphQL 服务用来向客户端推送数据。

2. Subscription 的 resolver 函数被一个对象包裹，并且需要作为 subscribe 字段的值。

下面，为了延续模块化结构的的 resolver 实现，新建一个名为 Subscription.js 的文件：

```js
touch src/resolvers/Subscription.js
```

下面是如何实现 subscription resolver 的代码：

```js
function newLinkSubscribe (parent, args, context, info) {
  return context.db.subscription.link(
    { where: { mutation_in: ['CREATED'] } },
    info,
  )
}

const newLink = {
  subscribe: newLinkSubscribe
}

module.exports = {
  newLink,
}
```

代码看上去很简洁。就如前面提到的，subscription resolver 作为 js 对象中， subscribe 字段的值。

context 中的 Prisma binding 实例也暴露了一个 subscription 对象作为 Prisma GraphQL API subscription 的代理。这个方法用来解析 subscription 并将数据推送给订阅了的客户端。Prisma 会处理实时函数内部复杂的逻辑。

打开 index.js 然后倒入 Subscription 模块：

```js
const Query = require('./resolvers/Query')
const Mutation = require('./resolvers/Mutation')
const AuthPayload = require('./resolvers/AuthPayload')
const Subscription = require('./resolvers/Subscription')
```

然后更新 resolver 对象的定义：

```js
const resolvers = {
  Query,
  Mutation,
  AuthPayload,
  Subscription,
}
```

## subscriptions 的测试

待续...






















