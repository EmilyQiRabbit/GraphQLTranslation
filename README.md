# [HOW TO GRAPHQL](https://www.howtographql.com/basics/0-introduction/) 中文版
  —— 在向着标准版和专业版努力的翻译

目前完成翻译的章节包括：

1. graphql 基础概念
2. 前端 React + Apollo
3. 后端 graphql + node

> 前端其他教程还包括 [React + urql](https://www.howtographql.com/react-urql/0-introduction/)；后端教程还包括：[graphql + elixir](https://www.howtographql.com/graphql-elixir/0-introduction/)，[graphql + ruby](https://www.howtographql.com/graphql-ruby/0-introduction/)，[graphql + java](https://www.howtographql.com/graphql-java/0-introduction/)，[graphql + python](https://www.howtographql.com/graphql-python/0-introduction/)，[graphql + scala](https://www.howtographql.com/graphql-scala/0-introduction/)。

[test 文件中 react-apollo 的官方源码请戳这里](https://github.com/howtographql/react-apollo)

Translated By Yuqi 🌸

🎉 欢迎校对者～

PS：最近本仓库正在持续的校对+更新中，fork 了的小伙伴记得要 fetch upstream 哦～～～

## 目录

* 基础教程
  * [基础教程 - GraphQL 简介](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/0-BasicTutorial/0-Introduction.md)
  * [GraphQL 是更好的 REST](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/0-BasicTutorial/1-GraphQL-is-the-better-REST.md)
  * [核心概念](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/0-BasicTutorial/2-Core-Concepts.md)
  * [架构](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/0-BasicTutorial/3-Big-Picture-Architecture.md)
* 高级教程
  * [客户端](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/1-AdvancedTutorial/0-Clients.md)
  * [服务端](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/1-AdvancedTutorial/1-Server.md)
  * [更多 GraphQL 概念](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/1-AdvancedTutorial/2-More-GraphQL-Concepts.md)
  * [GraphQL 工具和生态圈](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/1-AdvancedTutorial/3-Tooling-and-Ecosystem.md)
  * [安全性](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/1-AdvancedTutorial/4-Security.md)
  * [常见问题](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/1-AdvancedTutorial/5-Common-Questions.md)
* React - Apollo 教程
  * [React + Apollo 教程 - 简介](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/0-Introduction.md)
  * [React - Apollo 入门教程](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/1-Getting-Started.md)
  * [使用 query 加载新闻链接数据](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/2-Queries-Loading-Links.md)
  * [使用 mutation 创建新的新闻链接](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/3-Mutations-Creating-Links.md)
  * [路由配置](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/4-Routing.md)
  * [认证](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/5-Authentication.md)
  * [更多 mutation 请求实例，以及 store 的更新](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/6-More-Mutations-and-Updating-the-Store.md)
  * [条件过滤：查找符合条件的新闻链接列表](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/7-Filtering-Searching-the-List-of-Links.md)
  * [使用订阅（subscription）实现实时更新](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/8-Realtime-Updates-with-GraphQL-Subscriptions.md)
  * [分页](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/2-ReactApolloTutorial/9-Pagination.md)
* graphql - node 教程
  * [graphql-node 教程 - 简介](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/3-NodeJsTutorial/0-Introduction.md)
  * [GraphQL-NODE 入门](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/3-NodeJsTutorial/1-GettingStarted.md)
  * [Query 基础](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/3-NodeJsTutorial/2-ASimpleQuery.md)
  * [Mutation 基础](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/3-NodeJsTutorial/3-ASimpleMutation.md)
  * [为服务添加数据库](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/3-NodeJsTutorial/4-AddingADatabase.md)
  * [使用 Prisma 客户端，连接服务与数据库](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/3-NodeJsTutorial/5-ConnectingServerAndDatabasewithPrismaClient.md)
  * [认证](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/3-NodeJsTutorial/6-Authentication.md)
  * [GraphQL subscription 实时订阅](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/3-NodeJsTutorial/7-RealtimeGraphQLSubscriptions.md)
  * [条件查找、分页和排序](https://github.com/EmilyQiRabbit/GraphQLTranslation/blob/master/3-NodeJsTutorial/8-FilteringPaginationAndSorting.md)
