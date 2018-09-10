> * 原文地址：[Realtime GraphQL Subscriptions](https://www.howtographql.com/graphql-js/7-subscriptions/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 实时 GraphQL 订阅

在这一章节中，你将会学习如何通过实现 GraphQL subscriptions 为应用加入实时更新的功能。目标是通过 GraphQL 服务实现两个暴露给外部的的的订阅功能：

* 当一个新的 Link 被创建的时候为订阅用户推送实时更新。

* 当一个已存在的 Link 被点赞的时候为订阅用户推送实时更新。

## 什么是 GraphQL subscriptions？

订阅（subscriptions）是 GraphQL 的一项功能，它允许服务器在发生某些特定事件的时候，向客户端推送数据。订阅功能常常使用 WebSockets 实现。本篇配置中，服务器和参与订阅的客户端维持了一个持续的连接。它打破了“请求-回复”这样的之前所有 API 交互的模式。

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

下面，我们来写一写代码：实现一个允许客户端订阅新 Link 被创建信息的订阅函数。

就和 query 和 mutation 一样，第一步是去扩充 GraphQL schema 的定义。

打开应用的 schema 并添加 Subscription 类型：

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

## subscriptions 测试

做好了上面所有这些代码的准备，可以开始测试了。你可以通过同时使用 GraphQL Playground 的两个实例（窗口）来完成。

* 需要通过 ctrl+c 停止服务然后再次运行 node src/index.js

* 然后，打开两个浏览窗口并导航至地址：http://localhost:4000

你将要使用第一个 Playground 来发送订阅请求，并建立一个和服务端的永久性的 websocket 连接；用第二个来发送 post mutation 触发订阅。

在第一个 Playground 中，发送如下订阅：

```js
subscription {
  newLink {
    node {
      id
      url
      description
      postedBy {
        id
        name
        email
      }
    }
  }
}
```

和发送 query 以及 mutation 不同，发送 subscription 将不会看到及时的结果。相反，将会显示一个旋转的 loading 标识，表示服务器在等待事件的发生。

是时候触发 subscription 事件了。

在另一个 GraphQL Playground 中发送如下 post mutation。记得需要首先授权（授权的具体操作参见前一章）。

```js
mutation {
  post(
    url: "www.graphqlweekly.com"
    description: "Curated GraphQL content coming to your email inbox every Friday"
  ) {
    id
  }
}
```

现在，可以去看看发送了订阅的 Playground 中发生了什么？

## 添加投票功能

投票功能允许用户为特定的 link 点赞。第一步还是需要扩展代表投票的 Prisma 数据模型。

打开 database/datamodel.graphql 并调整为：

```js
type Link {
  id: ID! @unique
  createdAt: DateTime!
  description: String!
  url: String!
  postedBy: User
  votes: [Vote!]!
}

type User {
  id: ID! @unique
  name: String!
  email: String! @unique
  password: String!
  links: [Link!]!
  votes: [Vote!]!
}

type Vote {
  id: ID! @unique
  link: Link!
  user: User!
}
```

如你所见，你为数据模型添加了一个新的 Vote 类型。它和 User 类型以及 Link 类型都是是一对多的关系。

为了应用这些变化，并更新 Prisma GraphQL API，这样它才能包含 Vote 类型的增删改查操作。你需要重新部署服务。

```sh
prisma deploy
```

现在，已经了解了 schema 驱动的开发模式，下一步就是去扩展 schema 的定义，这样你的 GraphQL 服务就可以暴露出一个 vote mutation 的接口：

```js
type Mutation {
  post(url: String!, description: String!): Link!
  signup(email: String!, password: String!, name: String!): AuthPayload
  login(email: String!, password: String!): AuthPayload
  vote(linkId: ID!): Vote
}
```

Vote 类型依旧需要从 Prisma database schema 中引入：

```js
# import Link, LinkSubscriptionPayload, Vote from "./generated/prisma.graphql"
```

下一步，实现相关的 resolver 函数。

在 src/resolvers/Mutation.js 中添加如下函数：

```js
async function vote(parent, args, context, info) {
  // 1
  const userId = getUserId(context)

  // 2
  const linkExists = await context.db.exists.Vote({
    user: { id: userId },
    link: { id: args.linkId },
  })
  if (linkExists) {
    throw new Error(`Already voted for link: ${args.linkId}`)
  }

  // 3
  return context.db.mutation.createVote(
    {
      data: {
        user: { connect: { id: userId } },
        link: { connect: { id: args.linkId } },
      },
    },
    info,
  )
}
```

解释一下：

1. 和 post resolver 类似，第一步是通过 getUserId 函数认证请求的 jwt。如果认证成功，函数将会返回发起请求的用户的 userId，如果认证失败，函数将会抛出错误。

2. db.exists.Vote(...) 函数你可能会觉得陌生。Prisma binding 对象不仅仅暴露 Prisma database schema 中对应 query、mutation、subscription 的函数，也会对应每个 data modal 生成一个 exists 函数。exists 函数接受一个 where 过滤器参数，允许你定义该类型元素的特定的条件。当满足条件的元素在数据库中至少存在一个的时候，exists 函数将会返回 true。在上文的例子中，我们使用 exists 函数来保证发起请求的用户还没有为这个 Link 投票过。

3. 如果 exists 函数返回了 false，createVote 函数将会创建一个新的 Vote 元素，并和 User 以及 Link 实例相关联。

同时，别忘了在 export 语句中添加 vote resolver：

```js
module.exports = {
  post,
  signup,
  login,
  vote,
}
```

这一章最后的一个任务，就是为新建投票添加一个订阅 subscription。方法和 newLink 类似。

为应用 schema 的 Subscription 添加一个新的字段：

```js
type Subscription {
  newLink: LinkSubscriptionPayload
  newVote: VoteSubscriptionPayload
}
```

然后，从 Prisma API 的 GraphQL schema 中导入 VoteSubscriptionPayload：

```js
# import Link, LinkSubscriptionPayload, Vote, VoteSubscriptionPayload from "./generated/prisma.graphql"
```

最后，添加订阅的 resolver 函数：

```js
function newVoteSubscribe (parent, args, context, info) {
  return context.db.subscription.vote(
    { where: { mutation_in: ['CREATED'] } },
    info,
  )
}

const newVote = {
  subscribe: newVoteSubscribe
}
```

并更新 export 语句：

```js
module.exports = {
  newLink,
  newVote,
}
```

一切就绪，下面可以开始测试 newVote 订阅功能了。

[self Proofreading +1]
