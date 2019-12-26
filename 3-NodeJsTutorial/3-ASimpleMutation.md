> * 原文地址：[A Simple Mutation](https://www.howtographql.com/graphql-js/3-a-simple-mutation/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# Mutation 基础

在这一章，我们将会学习如何为 GraphQL API 添加 mutation。客户端通过该 mutation，能在服务端添加新的新闻链接。

## 扩展模式定义

和以前一样，首先需要在 GraphQL 模式定义中添加一种新的操作。

在 [index.js](https://github.com/howtographql/graphql-js/blob/master/src/index.js) 中，将 typeDefs 扩展为：

```js
const typeDefs = `
type Query {
  info: String!
  feed: [Link!]!
}

type Mutation {
  post(url: String!, description: String!): Link!
}

type Link {
  id: ID!
  description: String!
  url: String!
}
`
```

这时候，模式定义的代码已经有些长了。我们来重构一下代码，把模式定义放到一个单独的文件中：

在 src 目录新建一个文件并命名为 schema.graphql：

```sh
touch src/schema.graphql
```

接下来把所有 schema 定义放在这个新的 [schema.graphql](https://github.com/howtographql/graphql-js/blob/master/src/schema.graphql) 文件里：

```graphql
type Query {
  info: String!
  feed: [Link!]!
}

type Mutation {
  post(url: String!, description: String!): Link!
}

type Link {
  id: ID!
  description: String!
  url: String!
}
```

然后，我们再来整理 index.js 文件。

因为模式定义已经移动到了独立的文件里，所以可以删除 typeDefs 变量了。然后更新文件最下面 GraphQLServer 实例化代码：

```js
const server = new GraphQLServer({
  typeDefs: './src/schema.graphql',
  resolvers,
})
```

实例话 GraphQLServer 的参数 typeDefs 可以像之前一样是一个字符串，或者也可以是一个包含模式定义的文件地址（也就是像上面这段代码这样），这一点非常方便。

## 实现 resolver 函数

下一步就是为新字段实现其 resolver 函数。

更新 index.js 文件中的 resolver 函数为：

```js
let links = [{
  id: 'link-0',
  url: 'www.howtographql.com',
  description: 'Fullstack tutorial for GraphQL'
}]
// 1
let idCount = links.length
const resolvers = {
  Query: {
    info: () => `This is the API of a Hackernews Clone`,
    feed: () => links,
  },
  Mutation: {
    // 2
    post: (root, args) => {
       const link = {
        id: `link-${idCount++}`,
        description: args.description,
        url: args.url,
      }
      links.push(link)
      return link
    }
  },
}
```

首先，注意到我们已经移除了 Link 的 resolver 函数。我们并不需要它，因为 GraphQL 服务能够推断出 Link 的结构。

下面我们依旧按注释的数字依次讲解：

1. 我们添加了一个新的整型变量 idCount，用来为每个新的 Link 元素生成一个唯一的 id。

2. 这段代码是 post 字段 resolver 函数的实现，首先我们创建了一个新的 link 对象，然后把它添加到了已经存在的 links 列表中，最后返回这个新的 link 对象，它代表新建的新闻链接。

现在我们正好可以讨论下，resolver 函数的第二个参数 args。

这个参数携带了 API 操作的参数：在这个例子中就是新闻链接的的 url 和 description 字段。在之前的 feed 和 info 的 resolver 函数中我们并不需要它，因为模式定义中相应的字段没有定义任何参数。

## 测试 mutation

启动服务，然后测试一下这个新的 API 吧。你可以使用 Playground 发送下面这个简单的 mutation 请求：

```graphql
mutation {
  post(
    url: "www.prisma.io"
    description: "Prisma turns your database into a GraphQL API"
  ) {
    id
  }
}
```

服务端将会返回：

```graphql
{
  "data": {
    "post": {
      "id": "link-1"
    }
  }
}
```

随着发送次数的增加，idCount 也会跟着增加，返回的 id 将会变成 link-2，link-3 等等

为了验证 mutation 成功运行了，你可以发送 query feed 请求，服务将会返回所有你用 mutation 请求创建的新闻链接列表。

但是，当你重启服务后，之前添加的新闻链接信息就都没有了，因为这些信息仅存储于内存中。下一章我们将会学习如何为 GraphQL 服务添加数据库层，这样你就能永久的保存数据了。

## 练习

如果你想做更多关于 resolver 函数的练习，这里有一个小小的挑战。基于当前的实现，扩展 Link 类型的 GraphQL API 的完整增删改查（CRUD）功能。即，为如下的模式定义实现 resolver 函数：

```graphql
type Query {
  # Fetch a single link by its `id`
  link(id: ID!): Link
}

type Mutation {
  # Update a link
  updateLink(id: ID!, url: String, description: String): Link

  # Delete a link
  deleteLink(id: ID!): Link
}
```
