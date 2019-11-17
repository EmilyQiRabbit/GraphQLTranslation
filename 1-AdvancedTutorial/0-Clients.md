> * 原文地址：[Advanced Tutorial - Clients](https://www.howtographql.com/advanced/0-clients/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 高级教程 - 客户端

在前端使用 GraphQL API 是抽象客户端代码、实现函数化的好机会。例如，考虑下面这些 app 中经常需要的基础功能：

* 不构建 http 请求，直接发送 query 或者 mutation
* view 层整合
* 缓存
* 基于 schema 验证并优化 query

当然，不会有人阻止你直接使用 HTTP 来获取数据，然后自己做转化，直到数据正确的显示在 UI 上。但是 GraphQL 提供了抽象代码的能力能让你精简掉这些基础工作，让你能集中精力到那些真正重要的部分上去。下面，我们就来详细讨论这些任务是什么。

> 目前有两个主流的 GraphQL 客户端，一个是 [Apollo Client](https://github.com/apollographql/apollo-client)，它是社区驱动的，能为大部分开发平台提供灵活强大的 GraphQL 客户端。另一个是 [Relay](https://facebook.github.io/relay/)，它是 Facebook 旗下土生土长的 GraphQL 客户端，大力优化了性能但是只能在 web 端应用。

## 直接发送 Query 和 Mutation

GraphQL 的一个很大优点就是能够让你采用**声明式**的方法来获取数据或者更新数据。换句话说，我们在 API 抽象的梯子上又上了一步，也就根本不用关心底层的网络任务本身了。

你之前需要直接使用 HTTP 来从 API 加载数据，比如 JS 中的 fetch 方法或者 iOS 的 NSURLSession。但是使用 GraphQL 你只需要写好 query，在里面描述清楚对数据的要求然后让系统来负责发送请求并处理网络应答。这就是 GraphQL 客户端做的事情。

## View 层的整合 & 更新 UI

一旦服务器的应答被 GraphQL 客户端收到并处理，那些你请求的数据就需要以某种形式展现在 UI 上。UI 处理更新的方式取决于你的开发过程中采用了什么平台和框架，它们是不同的。

拿 React 来举例子，GraphQL 客户端采用了**高阶组件**的概念来获取需要的数据，然后作为 props 提供给组件。通常来说，GraphQL 的声明式特性和**函数驱动式编程**技术非常合，这两项技术联合起来非常强大， view 只需要声明它的数据依赖然后 UI 将会和你选择的 FRP 层绑定。

## 缓存 query 结果：概念和策略

大部分的应用都希望维护一份数据缓存，这份缓存能提供流畅的用户体验，并减轻用户获取数据消耗流量的负担。

通常情况下，当缓存数据时，开发者的第一直觉都是把这些从服务端获取的数据保存到一个本地的 store 中，之后在需要的时候可以直接访问。如果使用了 GraphQL，本地数据的访问就更简单了：把 GraphQL query 的结果直接放到 store 中，然后如果这个相同的 query 又一次被执行了，GraphQL 就会直接返回之前保存的数据。但是这个方法对于大部分应用效率都比较低（逗我？）。

更加高效的方法是提前规范数据格式。也就是将 query 的结果（可能包含对象的嵌套）展开，让 store 仅包含独立的数据，并且可以通过全局的唯一的 ID 查找。想了解更多的话，可以看看这篇 [Apollo 博文](https://dev-blog.apollodata.com/the-concepts-of-graphql-bc68bd819be3)。（大概浏览了下，感觉炒鸡棒 🎉 如果真的很好的话后续会翻译）

## 构建时的 Schema 验证 & 优化

Schema 包含了使用 GraphQL API 时客户端能够做的所有事的信息，那么在这里就有一个很好的机会在构建的时候就去验证和优化客户端想要发送的 query。

当构建环境到达 schema 的时候，它将解析项目中的所有 GraphQL 代码然后和 schema 中的信息对比。这就将能找出拼写错误和一些其他的错误，此时就发现错误要比应用到了用户手中才发现错误要好得多的多了.....

## 共置 view 视图和数据依赖

GraphQL 一个很强大的概念就是，它允许你并排使用 UI 和数据需求。视图和数据依赖之间的强耦合极大的提高了开发体验。消除了对于数据需要在 UI 界面哪个位置显示这部分编程的工作量。

共置能够多强大，取决于你的开发平台。例如在 JS 应用，将数据依赖和 UI 代码放在同一个文件中是可行的。在 Xcode 中，使用 Assistant Editor 可以用于同时处理视图控制器和 graphql 代码。