> * 原文地址：[A Simple Query](https://www.howtographql.com/graphql-js/2-a-simple-query/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 一个简单的 query

在这一章中，你将会实现你的第一个 API 操作，它为 Hacker News clone 网站提供的接口功能是：查询其他用户发起的 links。

## 扩展 schema 的定义

我们从实现一个 feed query 开始，它可以获取一个 List 元素的列表。一般情况下，当为 API 添加功能的时候，过程其实都差不多的哈～😁

1. 为 GraphQL schema 扩展一个新定义的 root field（如果需要，也要添加关联的数据类型，即 data types）
2. 为新增的 field 添加相关的 resolver。

这个过程也被叫做：schema-driven（schema 驱动）或者 schema-first（schema 优先）开发。

我们继续吧，来看第一步：GraphQL schema 扩展。

在 index.js 文件中，更新 typeDefs 常量：

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

很直观哈。新定义了一个 Link 类型，代表 Hacker News 可以新建的链接。每个 Link 都有 id, description, url 属性。接下来，为 Query 添加一个 root field，允许获取 Link 列表。这个列表不能是 null，并且也不能包含空元素 - 这是 [Link!]! 中 ! 的意义。

## 实现 resolver 函数

下一步是为 feed 请求实现 resolver 函数。事实上，有一件事情我们还一直没有提及：不仅仅是 root fields，而是所有 schema 的 field 都有 resolver 函数。所以你不仅需要为 Link 添加 resolver，也需要为 id, description 和 url field 添加。

在 index.js 中，添加一个假数据列表，然后更新 resolver：

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

我们根据注释的标号来看一看上面这段代码做了什么：

1. links 变量用来在运行时暂存 link 的信息。这些信息没有存储在数据库里，只能在运行时访问。
2. 为 root field 的 feed 添加 resolver。注意到，resolver 必须定义在在相关的 schema 定义后。例如上面这个例子中的 Query -> feed 结构。
3. 最后为 schema 中定义的 Link 类型添加另外三个 resolver。我们后面会讨论这里的入参 root 是什么。

启动服务然后测试一下吧。（停掉正在运行的服务，运行 node src/index.js）然后浏览器打开 http://localhost:4000。在 Playground 中，可以看到 feed 请求已经可用了！

试试看吧，发送请求：

```js
query {
  feed {
    id
    url
    description
  }
}
```

炒鸡棒，可以看到服务器发回了 links 列表：

```
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

可以删掉一些 query 中的 select set（也就是 id, url 啥的），看看服务器是如何返回数据的？

## query 的解析过程

现在我们简单介绍下，GraphQL 服务器是如何解析传入的请求的。正如你已经知道的，一个 GraphQL 请求包含一系列的 field，在 schema 中，它们都有对应的源。

重新审视下，这个请求：

```
query {
  feed {
    id
    url
    description
  }
}
```

所有字段：feed, id, url 和 description 都能在 schema 的定义中找到。现在你也知道了，schema 中的每个 field 背后都有一个 resolver 支持，这个 resolver 就负责精准返回这个 field 的数据。

现在你能想象解析过程是什么样的了吗？GraphQL 服务所要做的一切就是，触发请求中所有 field 对应的 resolver 函数，然后吧结果打包成请求需要的格式。请求解析现在就仅仅是编排 resolver 函数的触发过程。

下面，仅剩的问题就是 Link 类型的 resolver：

```js
Link: {
  id: (root) => root.id,
  description: (root) => root.description,
  url: (root) => root.url,
}
```

首先，每个 resolver 函数实际上都能接受 4 个参数。由于后三个对于上面的例子没用，所以就直接省略了。我们后面会介绍它们。

第一个参数，一般称为 root，代表的是上一层 resolver 执行的结果。这是什么意思呢？

我们知道，GraphQL 的请求是可以嵌套的。每一层嵌套也就对应了一层 resolver 执行的层。上面的那个 query 就有两层。

在第一层，触发了 feed resolver 函数，返回了 links 变量对应的那个数据。在第二层，GraphQL 服务很聪明，能够自动的触发 Link 类型中对应的每个 resolver，因为列表中每个元素都是在前一个 resolver 级别返回的，因此，在三个 Link 的 resolver 的每一个中，传入的 root 对象都是 Links 变量中的元素。

[戳这里有详细介绍](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e#9d03)

在这个例子中，Link resolver 其实完全可以省略。

[self Proofreading +1]
