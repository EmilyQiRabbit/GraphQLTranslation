> * 原文地址：[React + Apollo Tutorial - Introduction](https://www.howtographql.com/react-apollo/0-introduction/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# React + Apollo 教程 - 简介

提示：本教程的最终项目代码可以在 [Github](https://github.com/howtographql/react-apollo) 上找到。如果你在接下来章节的学习过程中遇到了困惑，那么随时可以参考它。同时提醒大家，每个代码块前面都标注有文件名，它直接链接到 GitHub 上对应的文件里，这样你就能清楚的知道当前代码应该处于什么位置，以及最终完成的时候它应该是什么样子。

## 概览

在前面的文章里，我们一起学习了 GraphQL 的主要概念和优势。现在是时候着手来完成一个实际的项目了！

我们将会试着复制 [Hackernews](https://news.ycombinator.com) 这个网站。这个应用将会有如下的功能：

* 展示一个新闻链接的列表
* 查询列表中的新闻链接
* 用户登录
* 登录用户可以创建新的新闻链接
* 登录用户可以帮顶新闻链接（每个用户对每条新闻只能帮顶一次）
* 当用户创建新的新闻链接或者帮顶时，要实时更新列表

搭建这个应用我们需要使用的技术是：

* 前端：
  * [React](https://reactjs.org)：搭建用户界面的前端框架
  * [Apollo Client 2.1](https://github.com/apollographql/apollo-client)：支持生产环境，同时支持缓存的 GraphQL 客户端

* 后端：
  * [graphql-yoga](https://github.com/prisma-labs/graphql-yoga)：功能全面的 GraphQL 服务端，它配置简单，性能优异，开发体验非常好
  * [Prisma](https://www.prisma.io)：开源的 GraphQL API 层，它能将数据库转化为 GraphQL API 的形式

我们使用命令 [create-react-app](https://github.com/facebook/create-react-app) 来创建 React 项目，这是一个使用广泛的命令行工具，它将提供给你一个空白的项目，并且已经设置好了所有必需的构建配置。

## 为什么选择 GraphQL 客户端？

在前面的 GraphQL 高级教程的“客户端”一章中，我们已经概括介绍了的 GraphQL 客户端的主要功能，下面我们再来更详尽一点的介绍它。

简单来说，由于你正在构建的应用中的存在某些重复的和结果不可知的任务，所以你应当使用 GraphQL 客户端。例如，我们在发送 query 和 mutation 请求时，可以完全不用担心底层网络细节或者维护本地缓存。这些都是和 GraphQL 服务交互的前端应用中的基本功能 -- 这里已经有一个搭建完成并且非常好用的 GraphQL 客户端了，你干嘛还非要自己再搭建一个呢？

有很多可用的 GraphQL 客户端库，它们可以提供给你对于 GraphQL 操作的不同的控制强度，也都有各自的长处和不足。对于非常简单的使用情境，[graphql-request](https://github.com/graphcool/graphql-request) 也就够用了。像这样的库都是封装发送到 GraphQL API 的 HTTP 请求的一个很薄的层。

但是，如果你想要写一个比较复杂的应用，需要包括缓存、更优的 UI 更新机制，还有其他的便捷功能等等。这时候你就需要一个功能完备的 GraphQL 客户端，让它帮助你处理 GraphQL 操作的生命周期。你可以选择 Apollo 客户端、Relay 或者 urql。

## Apollo、Relay 和 urql 三者的比较

当人们刚开始在前端使用 GraphQL 的时候，最常见的问题就是，应该选择哪一个 GraphQL 客户端呢？在这里，我们会提供给您一些提示，帮助您决定那个客户端才是最适合您手头的项目的。

[Relay](https://relay.dev) 是 Facebook 自己研发的 GraphQL 客户端，在 2015 年和 GraphQL 一起开源。它汇集了所有 Facebook 自 2012 年开始使用 GraphQL 后学到的所有经验。Relay 大幅度优化了性能，并且尽最大可能的降低了网络拥塞。一个有趣的小知识是，Relay 最一开始是作为一个路由框架开始研发的，但是最终将数据加载的职责也承担了起来。于是它演变成为了一个强大的数据管理解决方案，可以在 JavaScript 应用中用于和 GraphQL API 交互。

而 Relay 带来的性能提升要以很高的学习代价作为成本。Relay 是一个很复杂的框架，理解它所有的内容必需一段时间的深入研究。随着 1.0 版本，即 Relay Modern 的发布，情况有所改善，但是如果你刚开始学习 GraphQL，Relay 可能并不是一个正确的选择。

[Apollo 客户端](https://github.com/apollographql/apollo-client)是一个由社区驱动构建的一个易于理解、灵活并且强大的 GraphQL 客户端。Apollo 立志于构建一个可用于所有主流开发平台的库，可用于构建 web 端以及移动端应用。现在，它已经是一个可以和流行框架例如 React、Angular、Ember 或 Vue，以及一些早期版本的 iOS 和安卓客户端绑定的 JavaScript 客户端了。Apollo 可用于生产环境，并且有很多例如缓存、UI 优化、支持订阅等等便捷的功能。

[urql](https://github.com/FormidableLabs/urql) 则是一种更加动态方式，和 Relay 与 Apollo 相比，它是一个更新的项目。它更专注于 React，因此它的核心更专注于简单性和可扩展性。它具有构建高效 GraphQL 客户端的基准，同时你也可以通过““Exchanges”完全掌控它对 GraphQL 的操作和结果的处理方式。和 [@urql/exchange-graphcache](https://github.com/FormidableLabs/urql-exchange-graphcache) 结合在一起，它在功能上基本等同于 Apollo，但体积更小，同时 API 更有针对性。
