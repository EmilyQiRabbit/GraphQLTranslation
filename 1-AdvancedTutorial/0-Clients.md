> * 原文地址：[Advanced Tutorial - Clients](https://www.howtographql.com/advanced/0-clients/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 高级教程 - 客户端

前端使用 GraphQL API 是在客户端开发新的抽象代码、实现通用函数的大好时机。我们一起考虑一下如下这些应用通常都需要的基础功能：

* 不构建 http 请求，直接发送 query 或者 mutation
* view 层整合
* 缓存
* 基于模式的验证和 query 优化

你当然还是可以直接使用 HTTP 来获取数据，然后自己做数据转化，直到最后数据能正确的渲染在 UI 上。但是 GraphQL 有能力将很多基础代码抽象出来，省去了通常情况下研发过程中的体力活儿，从而让你能把精力集中到应用中那些真正重要的部分上。下面，我们就来详细讨论这些任务到底是什么。

> 目前有两个主流的 GraphQL 客户端，一个是 [Apollo Client](https://github.com/apollographql/apollo-client)，它是由社区推动的，能为大部分开发平台提供灵活强大的 GraphQL 客户端。另一个是 [Relay](https://facebook.github.io/relay/)，它是 Facebook 旗下土生土长的 GraphQL 客户端，它的性能被大幅度优化了，但是只能在 web 端应用。

## 直接发送 query 和 mutation

GraphQL 最大的优点之一，就是能够以**声明式**的方法来获取或更新数据。换句话说，我们可以进一步抽象 API，也不用亲自去写网络底层相关的代码了。

在 GraphQL 之前，你需要使用 HTTP 从 API 加载数据（比如 JS 中的 fetch 方法或者 iOS 的 NSURLSession），但如果时使用 GraphQL，你只需要写好 query，并在里面描述清楚对数据的要求即可，接下来 GraphQL 系统会负责发送请求并处理网络应答。这就是 GraphQL 客户端会为你做的事情。

## view 层整合与 UI 更新

一旦 GraphQL 客户端收到并处理了来自服务端的应答，请求到的数据就要以某种形式展现在 UI 上。UI 处理更新的方式都是不同的，它取决于你在开发过程中采用了什么平台和框架。

例如说 React，GraphQL 客户端采用了**高阶组件**的概念在后台获取需要的数据，然后作为属性，即 `props`，提供给组件。通常来说，GraphQL 的声明式特性和[**函数式反应式编程**](https://en.wikipedia.org/wiki/Functional_reactive_programming)（Functional reactive programming，FRP）技术很契合。这两项技术如果联合起来将会非常强大，即：view 层只需要声明它的数据依赖，然后 UI 就会和你选择的 FRP 层绑定。

## 缓存 query 结果：概念和策略

大部分的应用都希望维护一份来自服务端数据的缓存。本地缓存信息能提供流畅的用户体验，并减轻用户用于获取数据而消耗流量的负担。

通常情况下，当需要缓存数据时，开发者首先想到的就是把这些从服务端获取的数据保存到一个本地的 **store** 中，以便随后访问。而如果使用了 GraphQL，它提供的方法就更简单了：只要把 GraphQL query 的结果直接放到 store 中即可，随后如果相同的 query 又一次被执行，GraphQL 就会直接返回之前保存的数据。但其实这个方法对于大部分应用效率都比较低。

更加高效的方法是预先**规范**数据格式。也就是将 query 的结果（可能包含嵌套对象）展开，让 store 仅包含独立的数据，并且可以通过全局 ID 查找。想了解更多的话，可以看看这篇 [Apollo 博客](https://dev-blog.apollodata.com/the-concepts-of-graphql-bc68bd819be3)，它讲解得非常精彩。

## 构建时的模式（schema）验证与优化

由于模式包含了所有使用了 GraphQL API 的客户端的功能的信息，那么这就是一个可以在构建时验证并潜在地优化客户端想发送的 query 的大好时机。

当构建环境访问模式的时候，它可以解析项目中的所有 GraphQL 代码然后和模式中的信息对比。此时就能发现拼写错误或者一些其他的错误，此时就能发现错误，要比应用到了用户手中才发现错误的后果要轻得多。

## view 和数据依赖的结合

GraphQL 一个强大的概念是，它允许你并行使用 UI 和数据需求。视图和数据依赖之间的强耦合极大的提高了开发者的体验。消除了思考正确数据如何在 UI 界面正确位置显示的工作量。

这个结合能够多强大也取决于你的开发平台。例如在 JS 应用中，将数据依赖和 UI 代码放在同一个文件中是可行的。而在 Xcode 中，使用 **Assistant Editor** 就可以同时处理视图控制器和 graphql 代码。
