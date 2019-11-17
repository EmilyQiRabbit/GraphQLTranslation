> * 原文地址：[React + Apollo Tutorial - Introduction](https://www.howtographql.com/react-apollo/0-introduction/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# React + Apollo 教程 - 简介

提示：本教程最终的工程代码可以在 [Github](https://github.com/howtographql/react-apollo) 上找到。如果你在学习过程中感到困惑，随时可以参考它。并且每个代码块前面都有文件名，可以直接链接到 github 上对应的文件里，所以你能清楚的知道当前代码应该处于什么位置以及最终完成的时候它应该是什么样子。

## 概览

在前面的文章里，我们一起学习了 GraphQL 的主要概念和优势。现在是时候着手来完成一个实际的项目了！

你将会试着完成一份 [Hackernews](https://news.ycombinator.com) 的复制品～这个 app 将会有如下的功能：

* 展示一个新闻链接的列表
* 查询新闻链接
* 用户可以登录
* 登陆用户可以创建新的链接
* 登陆用户可以帮顶链接
* 当用户创建新的链接或者帮顶的时候要实时更新列表

搭建这个 app 采用的技术是：

* 前端：
  * React：搭建用户界面的前端框架
  * Apollo Client：生产就绪，高速缓存 GraphQL 客户端（*原文：Production-ready, caching GraphQL client，不知这么翻译是否合适*）

* 后端：
  * graphql-yoga：功能全面的 GraphQL Server，着重于设置简单，性能优异以及卓越的开发人员体验
  * Prisma：开源的 GraphQL API 层，能将数据库接口转化为 GraphQL API

我们使用 [create-react-app](https://github.com/facebook/create-react-app) 来创建 React 项目，这是一个流行的命令行工具，它将提供一个空白的项目，并且已经安装了所有必需的构建配置。

## 为什么选择 GraphQL 客户端？

在 GraphQL 基础部分的 Client 章节中，我们已经概括介绍了的 GraphQL 客户端的主要功能，下面我们更进一步。

简单来说，由于你正在构建的 app 中的存在某些重复的和结果不可知的任务，所以你应当使用 GraphQL 客户端。例如，发送 query 和 mutation 请求时，完全不用担心底层的网络细节或者维护本地缓存。这是和 GraphQL 服务交互的前端应用中的基本功能 -- 已经有了非常好用的 GraphQL 客户端，你干嘛还要自己搭建一遍呢？

有很多可用的 GraphQL 客户端库。对于非常简单的使用情境，[graphql-request](https://github.com/graphcool/graphql-request) 也许就够用了。但是，如果你想要写一个比较复杂的应用，需要包括缓存、优化 UI 更新还有其他的便捷功能等等，事情就不一样了。这时，你可以选择 Apollo Client 或者 Relay。

## Apollo vs Relay

...待续...

[self Proofreading +1]
