> * 原文地址：[Node.js Tutorial - Introduction](https://www.howtographql.com/graphql-js/0-introduction/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# graphql-node 教程 - 简介

> 注：本教程最终代码可以在 [GitHub](https://github.com/howtographql/graphql-js) 上查看，如果你在接下来章节的学习过程中遇到了困惑，那么随时可以参考它。同时提醒大家，每个代码块前面都标注有文件名，它直接链接到 GitHub 上对应的文件里，这样你就能清楚的知道当前代码应该处于什么位置，以及最终完成的时候它应该是什么样子。

## 概览

在后台技术中，GraphQL 可算得上是一颗冉冉升起的新星。它也是一种 API 设计规范，并可以替代 REST，现在正逐渐成为用来暴露服务端数据和功能的新标准。

在 graphql-node 这部分教程中，我们将学习如何从零开始，完整建立一个比较通用的 GraphQL 服务。我们会用到的技术包括：

* [`graphql-yoga`](https://github.com/prisma/graphql-yoga)：功能全面的 GraphQL 服务，其配置简单，同时具有高性能和极佳的开发体验。它基于 [Express](https://expressjs.com)，[apollo-server](https://github.com/apollographql/apollo-server) 和 [graphql-js](https://github.com/graphql/graphql-js) 等。

* [Prisma](https://www.prisma.io)：用于替代传统 ORM（Object/Relational Mapping，即对象-关系映射）。可使用 Prisma 实现 GraphQL 的 resolver 函数，简化数据库访问。

* [GraphQL Playground](https://github.com/prisma-labs/graphql-playground)：它是 “GraphQL IDE”，可以通过发送 query 和 mutation 请求对 GraphQL API 的功能进行交互性测试。它和可以测试 REST API 的 Postman 有些类似。但是 GraphQL Playground 还有更多功能，包括：

  * 自动为所有可用 API 生成一份全面的文档。
  * 提供编辑器，你可以在里面写入 query，mutation 和 subscription，同时还有自动填充和语法高亮功能。
  * 便捷共享 API 操作。

## 目标

本教程的目的是为前端应用：复制版 [Hacker News](https://news.ycombinator.com) 提供 API。以下是本教程内容简要说明。

我们从 GraphQL 服务运行的基础开始学习，包括定义服务的 [GraphQL 模式（schema）](https://www.prisma.io/blog/graphql-server-basics-the-schema-ac5e2950214e)，以及编写相关的 resolver 函数。刚开始的时候，这些 resolver 只处理保存在内存中的数据 - 所以在服务停止运行的时候数据将被销毁。

单没人希望自己的服务无法保存并维护数据，所以你还需要为系统添加一个数据库层。数据库层由 Prisma 驱动，通过 [Prisma 客户端](https://www.prisma.io/docs/prisma-client)与 GraphQL 服务连接。

一旦连接了数据库，你就可以为 API 添加更多高级功能。

我们将从实现登录注册功能开始，让用户能够使用 API 进行身份验证。这也将允许你检查用户对某些 API 操作的权限。

教程的下一个部分是关于使用 GraphQL subscription 为 API 添加实时更新的功能。

最后，我们将支持条件查询和分页功能，这就允许发起 API 请求的客户端可以限制从 API 返回的项目列表。

现在就开始吧 🚀
