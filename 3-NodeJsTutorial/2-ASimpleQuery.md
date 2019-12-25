> * 原文地址：[A Simple Query](https://www.howtographql.com/graphql-js/2-a-simple-query/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# Query 基础

在本篇教程中，我们将会实现第一个 API 操作，它可以为克隆版的 Hacker News 网站提供可以用来查询户创建的新闻链接的接口。

## 扩展模式定义

我们从实现 feed query 开始，这个接口可以返回 List 类型元素的列表。一般情况下，为 API 添加新功能的过程都很相似：

1. 为 GraphQL 模式定义扩展新的根字段（如果需要，也要添加相关联的数据类型）

2. 为新增字段添加与之相关的 resolver 函数。

这个过程也称为模式驱动（schema-driven）或者模式优先（schema-first）开发。

下面我们来完成第一步：扩展 GraphQL 模式定义。

在 [index.js](https://github.com/howtographql/graphql-js/blob/master/src/index.js) 文件中，更新 typeDefs 常量：

```js
const typeDefs = `
type Query {
  info: String!
  feed: [Link!]!
}

type Link {
  id: ID!
  description: String!
  url: String!
}
`
```

代码简单直观：新定义了 Link 类型，它代表 Hacker News 应用中的新闻链接。每个 Link 都有 id, description 和 url 属性。接下来，为 Query 添加一个根字段 feed，允许用户获取 Link 元素的列表。这个列表不能是空，并且也不能包含空元素 —— 这是 `[Link!]!` 中两个 `!` 的作用。

## 实现 resolver 函数

下一步是实现 feed 对应的 resolver 函数。事实上有一点我们还未曾提及：不仅仅是根字段，GraphQL 模式中所有字段都有与之相关的 resolver 函数。所以你不仅需要为 Link 类型添加 resolver 函数，也要为 id, description 和 url 字段都添加 resolver 函数。

在 index.js 中，添加一个假的数据列表 links，然后更新 resolver：

```js
// 1
let links = [{
  id: 'link-0',
  url: 'www.howtographql.com',
  description: 'Fullstack tutorial for GraphQL'
}]

const resolvers = {
  Query: {
    info: () => `This is the API of a Hackernews Clone`,
    // 2
    feed: () => links,
  },
  // 3
  Link: {
    id: (root) => root.id,
    description: (root) => root.description,
    url: (root) => root.url,
  }
}
```

我们按照注释中的标号依次解释这段代码：

1. links 变量用来在运行时暂存新闻链接信息。目前这些信息没有存储在数据库里，只能在运行时访问。

2. 为根字段 feed 添加 resolver 函数。我们注意到，resolver 函数必须以模式定义中的相应字段命名。

3. 最后为模式定义中的 Link 类型添加另外三个 resolver。我们后面会讨论这里的参数 root 是什么。

启动服务然后测试一下吧,（首先使用 CTRL+C 停止正在运行的服务，然后再次运行 node src/index.js）在浏览器打开` http://localhost:4000`。于此同时，在 Playground 中，可以看到 feed 请求已经可用了！

试试看吧，发送请求：

```graphql
query {
  feed {
    id
    url
    description
  }
}
```

炒鸡棒，可以看到服务端返回了 links 变量对应的列表：

```graphql
{
  "data": {
    "feed": [
      {
        "id": "link-0",
        "url": "www.howtographql.com",
        "description": "Fullstack tutorial for GraphQL"
      }
    ]
  }
}
```

可以删掉 query 中可选集合的字段（id，url 或者 description），看看服务端返回的数据有什么变化。

## query 的解析过程

现在我们简单介绍一下，GraphQL 服务器是如何解析请求的。如你所见，GraphQL 请求会包含一些字段，而这些字段在 GraphQL 模式的类型定义中，都有对应的源。

我们重新思考这个请求：

```graphql
query {
  feed {
    id
    url
    description
  }
}
```

所有的 4 个字段：feed, id, url 和 description 都能在 GraphQL 模式定义中找到。我们前面提到过，模式中的每个字段都有一个 resolver 函数支持，该 resolver 函数就负责返回这个字段对应的的数据。

现在你能想象出解析过程是什么样的了吗？GraphQL 服务所要做的一切就是，触发请求中所有 field 对应的 resolver 函数，然后吧结果打包成请求需要的格式。请求的解析其实就是按需触发 resolver 函数的过程。

下面，仅剩的问题就是 Link 类型的 resolver 函数，它们似乎都遵循一个简单的模版：

```graphql
Link: {
  id: (parent) => parent.id,
  description: (parent) => parent.description,
  url: (parent) => parent.url,
}
```

首先很重要的一点是，实际每个 resolver 函数都能接受 4 个参数。由于后三个目前我们还不需要，所以暂时省略。别担心，在后续的文章中我们很快就会学到。

第一个参数，一般称为 `parent`（或者 `root`），它是上级 resolver 函数执行的结果。那这又意味着什么？🤔

我们知道，GraphQL 请求是可以嵌套的。每一层的嵌套都就对应着一个 resolver 函数执行层。因此，上面的那个 query 是两层的。

第一层触发了 feed resolver 函数，返回了 links 变量对应的数据。在第二层，GraphQL 服务能够自动的触发 Link 类型对应的 resolver（这是由于有了模式，服务知道 feed 应该返回 Link 类型的元素），因为列表中每个元素都是上一层 resolver 函数返回的，因此，这三个 Link 类型 resolver 函数中，传入的 parent 对象就是 links 列表中的元素。

> [戳这里有详细介绍](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e#9d03)。

在这个例子中，Link 中的 resolver 函数其实完全可以省略 👌。去掉它们也不会影响服务的运行。
