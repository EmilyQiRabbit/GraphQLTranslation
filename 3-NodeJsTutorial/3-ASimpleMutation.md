> * 原文地址：[A Simple Mutation](https://www.howtographql.com/graphql-js/3-a-simple-mutation/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 一个简单的 Mutation

这一部分你将会学习如何为 GraphQL API 添加一个 mutation。这个 mutation 的功能是允许以 post 方法在服务端添加一个新的 link。

## 扩展 schema

和以前一样，首先需要在 GraphQL schema 定义中添加一个新的操作。

在 index.js 中，添加类型定义：

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

这时候，其实 schema 的定义代码已经变得有些长了。我们来稍微重构下代码，吧 schema 放到一个单独的文件中：

在 src 目录新建一个文件并命名为 schema.graphql：

```
touch src/schema.graphql
```

接下来把所有 schema 定义放在这个新的文件里：

```js
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

有了这个文件，index.js 文件就可以变得整齐一些了～

因为 schema 已经在新的文件里，所以可以删除 typeDefs 变量了。然后更新 GraphQLServer 的实例化代码：

```js
const server = new GraphQLServer({
  typeDefs: './src/schema.graphql',
  resolvers,
})
```

很方便的一点是，GraphQLServer 实例化参数中，typeDefs 可以像之前一样用一个字符串来提供，或者也可以用一个包含 schema 定义的文件来提供。

## 实现 resolver 函数

为 API 添加新的功能的下一步就是为新的 field 实现 resolver 函数。

更新 resolver 函数：

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

首先，注意到我们已经移除了 Link resolver，我们并不需要他们，因为 GraphQL 服务能够推断出 Link 的结构。

下面我们按照注释的数字逐个讲解：

1. 添加了一个新的整型变量，为的是为每个新的 link 生成一个唯一的 id。

2. post resolver 的实现中，首先创建了一个新的 link 对象，然后把它添加到了已经存在的 links 列表中，最后返回了这个新的 link。

现在，我们正好可以讨论下，resolver 函数的第二个参数 args。

这个参数携带了操作的参数：在这个例子中就是 Link 的 url 和 description。在之前的 resolver 中，我们并不需要它，因为相应的字段没有在 schema 的定义中指定任何参数。

## 测试 mutation

启动服务，然后测试新的 API 吧。下面这是一个简单的 mutation 操作，你可以用 Playground 发送：

```js
mutation {
  post(
    url: "www.prisma.io"
    description: "Prisma turns your database into a GraphQL API"
  ) {
    id
  }
}
```

服务将会返回：

```js
{
  "data": {
    "post": {
      "id": "link-1"
    }
  }
}
```

随着发送次数的增加，idCount 也增加，返回的 id 将会变成 link-2, link-3...

问了验证 mutation 成功运行了，你可以发送 seed 请求，服务将会返回所有你用 mutation 创建的 link。

但是，当你重启服务后，之前你添加的 link 就都没有了，因为这些 link 就仅在服务运行时存在于内存中。在下一节中，你将会学习如何为 GraphQL 服务添加数据库层，这样你就能持久化你的变量了。

## 练习

如果关于 resolver 函数，你想做更多的练习，这里有一个小小的挑战可以给你。基于现在的实现，将关于 link 的 GraphQL API 扩展成为完整的 CRUD（增删改查）功能。也就是实现如下的 schema 定义：

```js
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