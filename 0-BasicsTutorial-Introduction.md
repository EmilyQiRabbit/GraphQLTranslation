> * 原文地址：[Basics Tutorial - Introduction](https://www.howtographql.com/basics/0-introduction/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 基础教学 - GraphQL 简介

GraphQL 是一个新的接口规范，相比于之前的接口规范例如 REST，GraphQL 更加高效、更加强大也更加灵活。它由 Facebook 开发，并且开源。现在它由一个大社区来维护，社区成员是来自全世界的个人和公司。

> 在软件开发的基础架构中，API 无处不在。简单来说，API 定义了**客户端**如何从**服务器**获取数据。

GraphQL 的精髓在于 -- 声明式的数据获取（declarative data fetching）。也就是，客户端能够精确指定，它需要从 API 中获取什么样的数据。和那些只能返回固定的数据结构的服务器接口端不同，GraphQL 服务仅仅暴露一个接口，它能够准确返回客户端索求的数据格式。

## GraphQL - 接口请求语言

现在，大多数应用都需要从服务端获取数据（因为仅有服务端有数据库嘛）。对于 API 来说，它们的任务就是提供应用所需的数据接口。

很多人经常会吧 GraphQL 和数据库技术混淆。这肯定是对 GraphQL 的误解了 -- GraphQL 是一种接口请求语言，而不是数据库请求语言。GraphQL 可以和任何数据库配合使用。而且不管在哪里，只要上下文语境需要 API，GraphQL 就都能使用，同时将大大提高 API 的效率。

## GraphQL 是 REST 的高效替换方案

> [这篇博客](https://blog.graph.cool/top-5-reasons-to-use-graphql-b60cfa683511)罗列了使用 GraphQL 的五个最重要优势，可以作为拓展阅读（需要全局模式科学上网 🤣）。

REST 是一种很流行的从服务器获取数据的方法。当 REST 的概念最初被创立的时候，客户端应用还相对简单，发展速度和迭代速度也远不如现在。因此，那时 REST 就是一个适应性比较强的解决方案。但是！但是哈，在过去的几年里，API 发展迅速变化巨大。特别是，有三个因素正在对传统的 API 发起挑战：

### 1、由于移动端应用需求增加，需要更高效的获取数据。

激增的移动端应用，并且设备电量有限、网络受限，这些都是 Facebook 最初为什么要设计 GraphQL。GraphQL 将需要网络传输的数据缩减到最小，这最大程度的优化了移动端的应用。

### 2、各式各样花里胡哨各不相同的前端框架和平台

由于客户端前端的框架和平台都各不相同，这就使得建立并维护一个通用的 API 很困难。但是用了 GraphQL，每个客户端都可以精确的获取它想要的数据。

### 3、迭代迅速 & 期望快速的功能开发

对于许多公司，持续开发是标配 -- 因为快速的迭代和高频的产品更新必不可少。如果用的是 REST API，服务器端的数据接口就需要根据客户端的需求和设计的变化而频繁修改。这对产品的快速迭代和发展是有一定的阻碍的。

## GraphQL 的历史、使用情景和被采用的情况

### GraphQL 并不仅仅适用于 React 开发者

Facebook 从 2012 年开始在原生的移动端应用 GraphQL。但是很有意思的是，GraphQL 却主要被应用在了 web 技术中，在原生的移动端反而被应用的很少。

Facebook 第一次公开谈论 GraphQL 是在 2015 年 React.js 大会上，然后就很快公布将会开源。因为 Facebook 总是习惯于在 React 的背景下谈论 GraphQL，所以对于非 React 的开发者，就总是误解 GraphQL 只能应用于 React，但其实并不是。

### 快速发展的社区

实际上，GraphQL 技术能够被应用于所有需要 API 的客户端。Netflix 和 Coursera 也在开发类似的应用，好让 API 接口更加高效。Coursera 的设想是研发某种让客户端制定数据需求的技术，Netflix 甚至已经开源了他们的解决方案：Falcor。而 GraphQL 开源之后，Coursera 则彻底取消了他们自己的项目，蹦蹦跳跳的就去测试 GraphQL 了。🤣

现在，GraphQL 已经被很多公司的产品应用了，比如 GitHub，Twitter，Yelp 和 Shopify。
