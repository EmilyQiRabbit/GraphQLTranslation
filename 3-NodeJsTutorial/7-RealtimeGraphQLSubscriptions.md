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



待续...