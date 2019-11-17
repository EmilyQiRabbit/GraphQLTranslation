> * 原文地址：[Basics Tutorial - Introduction](https://www.howtographql.com/basics/0-introduction/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 基础教程 - GraphQL 简介

GraphQL 是一种可以替代 REST 的新的接口规范，它更加高效、强大，也更加灵活。GraphQL 由 Facebook 开发，并已经[开源](https://reactjs.org/blog/2015/02/20/introducing-relay-and-graphql.html)。目前，来自世界各地的公司和个人组成的社区，都在共同维护 GraphQL。

> 在软件开发的基础架构中，API 是无处不在的。简单来说，API 定义了**客户端**如何从**服务端**获取数据。

GraphQL 的核心在于**声明式数据获取（declarative data fetching）**，即客户端能够精确指定从 API 中获取什么数据。和大多数只能返回固定数据结构的接口不同，GraphQL 服务仅仅暴露一个接口，并能准确返回客户端要求的数据格式。

## GraphQL 是 API 的请求语言

现在，几乎所有的应用都需要从服务端获取数据（因为仅有服务端有数据库嘛）。对于 API 来说，它们的任务就是提供应用所需的数据接口。

很多人经常会认为 GraphQL 是一种数据库技术。这肯定是一种错误的观念了 —— GraphQL 是一种 API 的请求语言，而不是数据库请求语言。如此说来，GraphQL 与数据库并无关系，它可以在任何使用了 API 的情景中有效的使用。

## GraphQL 是可替代 REST 的更高效规范

> [这篇博客](https://blog.graph.cool/top-5-reasons-to-use-graphql-b60cfa683511)罗列了使用 GraphQL 最主要的 5 个优势，可以作为拓展阅读。

REST 是一种服务器用来公开数据的流行方法。当 REST 这个概念最初被创立的时候，客户端应用还相对简单，发展速度和迭代速度也还远不如现在。因此，那时的 REST 就是一个适应性比较强的解决方案。但是在过去的几年里，API 发生了巨大的变化。这其中，有三个最主要的因素正在对传统 API 的设计方式发起挑战：

### 1、移动端应用用量增加，需要更高效的获取数据。

移动端应用激增，而移动设备电量有限、网络受限，这些都是 Facebook 最初设计 GraphQL 的原因。GraphQL 可以将网络传输的数据缩减到最小，这极大程度的优化了移动端的应用。

### 2、各不相同的前端框架和平台

运行客户端应用的前端框架和平台各不相同，这就使得构建并维护一个通用的 API 很困难。但是用了 GraphQL，所有客户端都可以精确获取它们想要的数据。

### 3、迭代迅速 & 渴望快速研发新特性

持续化部署对于许多公司来说都是基本配置，快速的迭代和高频的产品更新是必不可少。如果公司使用 REST APIs，服务端的数据接口就需要根据客户端的需求和设计变化而频繁的修改。这对产品的快速发展和迭代是有一定的阻碍的。

## GraphQL 的历史、背景和应用

### GraphQL 不仅为 React 开发者而生

Facebook 从 2012 年开始在他们的原生移动端应用 GraphQL。但是很有意思的是，后来 GraphQL 却主要被应用在了 web 技术中，而在原生的移动端仅吸引了很少的用户。

Facebook 第一次公布 GraphQL 是在 2015 年 [React.js 大会](https://www.youtube.com/watch?v=9sc8Pyc51uU)上，随后很快公布，[会将 GraphQL 开源](https://reactjs.org/blog/2015/05/01/graphql-introduction.html)。由于 Facebook 的工程师总是习惯于在 React 的背景下讨论 GraphQL，所以对于非 React 的开发者，就很容易误解为：GraphQL 只能应用于 React，但其实并不是。

### 发展迅速的社区

事实上，GraphQL 技术能够被应用于所有需要 API 的客户端。有趣的是，其他公司例如 Netflix 和 Coursera 也在开发类似的应用，好让 API 接口更加高效。Coursera 的设想是研发某种让客户端制定数据需求的技术，Netflix 甚至已经开源了他们的解决方案：Falcor。而在 GraphQL 开源之后，Coursera 却彻底取消了他们自己的项目，转而上了 GraphQL 的车。

现在，GraphQL 已经被很多公司应用于生产环境了，比如 GitHub、Twitter、Yelp 和 Shopify 等等。

更多资源可见：[GraphQL Conf](https://graphqlconf.org)，[GraphQL 每周新闻](https://graphqlweekly.com)
