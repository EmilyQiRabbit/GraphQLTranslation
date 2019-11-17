> * 原文地址：[Node.js Tutorial - Introduction](https://www.howtographql.com/graphql-js/0-introduction/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# Node.js 教程 - 简介

> 注：教程的最终代码在 [GitHub](https://github.com/howtographql/graphql-js) 上，当你在下面章节的课程学习中感到困惑的时候，可以用它作为参考。同时还要注意，每个代码块都有一个文件名注释，这个注释可以直接链接到 GitHub 上相关源码文件，方便你查看当前代码的位置以及代码的最终形态。

## 概览

在后台技术中，GraphQL 可以算一颗冉冉升起的新星。它作为一个 API 设计规范，可以替代 REST，正逐渐成为用来暴露服务器数据和功能的新标准。

在本教程中，你将会从零学习如何完整建立一个一般的 GraphQL 服务。我们将会用到如下技术：

* graphql-yoga：功能全面的 GraphQL 服务，专注于简单配置，高性能和高开发体验。建立在 Express，apollo-server，graphql-js 等之上。

* Prisma：GraphQL 数据库代理，它可以将你的本地数据库转化为 GraphQL API。这个 API 能为你的数据模型提供强大的实时增删改查操作。

* graphql-config & GraphQL CLI：优化 GraphQL 相关工作流的工具。

* GraphQL bindings：处理 GraphQL API 的简便方法。binding 可以为每个 API 操作生成专用的 JavaScript 函数。

* GraphQL Playground：它是 “GraphQL IDE”，允许通过发送请求和修改操作来互动测试 GraphQL API 的功能。有点像 Postman 测试 REST APIs。除了其他事项外，GraphQL Playground 还可以：

  * 自动生成所有可用 API 操作的一份很全面的文档。
  * 提供了一个编辑器，你可以在里面写入 queries，mutations，subscriptions，同时还有自动填充和语法高亮。
  * 允许你方便的共享 API 操作。

## 目标

本教程的目的是为前端应用 Hacker News 的克隆版创建一个 API。以下是本教程目标内容的简要说明。

你将从学习 GraphQL 服务如何运行的基础开始学习，包括定义服务的 GraphQL schema 以及编写相关的 resolver 函数。一开始的时候，这些 resolver 将只处理内存中保存的数组 - 所以在服务停止运行的时候数据都不存在了。

没人希望自己的服务没办法保存并维持数据，所以你还需要添加一个数据库层。数据库层由 Prisma 驱动，并且将会通过 Prisma bindings 链接你的 GraphQL server。你可以认为这些 binding 就是帮你妥善处理请求的 “GraphQL ORM（Object Relational Mapping - 对象关系映射）”。

一旦连接了数据库链接，你将可以为 API 添加更多高级功能。

你将从实现登录注册功能开始，使用户能够根据 API 进行身份验证。这也将允许您检查用户对某些 API 操作的权限。

教程的下一个部分是关于使用 GraphQL subscriptions 为 API 添加实时功能。

最后，你将允许 API 的请求者限制他们从 API 取回的项目列表，方法是添加过滤条件和分页能力。

让我们开始吧！ 🚀

[self Proofreading +1]