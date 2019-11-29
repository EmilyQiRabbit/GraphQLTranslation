> * 原文地址：[Common Questions](https://www.howtographql.com/advanced/5-common-questions/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 常见问题

## GraphQL 是一种数据库技术吗？

非也。人们常常会把 GraphQL 和数据库技术搞混在一起。这绝对是一个错误的观念，GraphQL 是一种 API 请求语言 —— 而不是数据库语言。所以说，其实它可以和任何数据库配合使用，甚至是没有数据库都可以。

## GraphQL 只适用于 React 或 Javascript 开发者吗？

非也。GraphQL 是一种 API 技术，所以它可以应用于任何需要 API 的场景。

后端的 GraphQL 服务可以使用任何能构建 web 服务的语言实现。除了 Javascript，还有很流行的基于 Ruby、Python、Scala、Java、Clojure、Go 和 .NET 的实现方式。

由于 GraphQL API 通常要通过 HTTP 进行操作，所以任何能与 HTTP 交互的客户端都可以从 GraphQL 服务请求数据。

> 注意：实际上 GraphQL 可以和任何传输层配合使用，所以你也可以选择 HTTP 之外的协议来实现服务。

## 如何做服务端缓存？

当我们把 GraphQL 和 REST 做对比的时候，一个很常见的顾虑就是 GraphQL 难以维护服务端缓存。而如果是使用 REST，由于接口数据结构不会改变，所以为每个接口缓存数据是非常容易的。

而如果使用了 GraphQL，我们并不知道客户端下一个请求是什么结构，所以为 API 配置一个缓存层其实没有必要。

服务端缓存现在仍旧是 GraphQL 的一大挑战。更多关于此的信息可以在 [GraphQL 官网](https://graphql.org/learn/caching/)查看。

## 如何认证和授权？

认证和授权的概念经常被混淆。认证描述的是身份声明的过程。当你使用用户名和密码登录服务的时候，就是认证自身的过程。另一方面，授权则描述的是权限规则，这些规则表明了用户和群组是否有权利获取系统特定的功能。

GraphQL 的认证可以使用例如 OAuth 这样的通用模式实现。

而关于授权的实现，[官方建议](https://graphql.org/learn/authorization/)是将所有的数据访问逻辑都委托给业务逻辑层，而不要在 GraphQL 的实现中直接处理。如果你想要了解更多关于授权的实现，可以看看 [Graphcool 的权限规则](https://www.graph.cool/docs/reference/auth/overview-ohs4aek0pe)。

## 如何处理错误？

一个成功的 GraphQL 请求应当返回一个根字段名为“data”的 JSON 对象。而如果请求失败或者是部分失败了（例如，用户没有权限获取这部分数据），那么在返回值中会有第二个名为“errors”的根结点：

```graphql
 {
  "data": { ... },
  "errors": [ ... ]
}
```

更多内容可以参见 [GraphQL 规范](https://graphql.github.io/graphql-spec/)。

## GraphQL 支持离线使用吗？

GraphQL 是用于网络 API 的语言，从这个定义的角度来说它仅支持在线使用。但是，客户端对离线使用的支持也是必须考虑的。Relay 和 Apollo 的缓存功能可能已经足够这些场景使用了，但是目前还并没有一个流行的解决方案能在本地永久保存数据。你可以在 GitHub 上找到关于 Relay 和 Apollo 并且在讨论离线支持的 issue 来获取更多信息。

> [这里](http://www.east5th.co/blog/2017/07/24/offline-graphql-queries-with-redux-offline-and-apollo/)介绍了一个关于离线应用和持久化的很有趣的方法。
