> * 原文地址：[Connecting Server and Database with Prisma Bindings](https://www.howtographql.com/graphql-js/5-connecting-server-and-database/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 用 Prisma Binding 链接服务和数据库

在这一章，你将学到如何连接 GraphQL 服务和提供数据库接口的 Prisma 数据库服务。连接是通过 Prisma Binding 完成的。

## 更新 resolver 函数以使用 Prisma Binding

首先你需要做一些清理和重构的工作。

打开 index.js 然后移除 links 数组和 idCount 变量 -- 你不再需要这些了，因为他们将会被保存在数据库中。

下一步，需要更新 resolver 函数，因为它们现在需要访问并不存在的变量了。而且，你现在希望数据是从数据库返回的，而不是本地保存的假数据。

在文件 index.js 中，更新 resolvers 对象的代码。

```js
const resolvers = {
  Query: {
    info: () => `This is the API of a Hackernews Clone`,
    feed: (root, args, context, info) => {
      return context.db.query.links({}, info)
    },
  },
  Mutation: {
    post: (root, args, context, info) => {
      return context.db.mutation.createLink({
        data: {
          url: args.url,
          description: args.description,
        },
      }, info)
    },
  },
}
```

哇，看上去好奇怪，因为有很多新的东西出现了，我们现在来看看它们都是做什么的？从 feed resolver 开始。

### context 和 info 参数

之前的 feed resolver 没有任何参数 -- 但是现在参数有四个了。事实上，前两个参数并不需要，需要的是 context 和 info。

还记得我们之前说过的：所有的 GraphQL resolver 函数都会接受四个参数。现在我们也知道了后两个 -- 但是它们是用来做什么吗的呢？

context 参数是一个 JS 对象，resolver 链上的所有 resolver 都能对它进行读出和写入操作 -- 因此它也就基本上成为了 resolver 之间数据交流的方法。你将会看到，当 GraphQL 服务本身被初始化的时候，context 就可以被写入了。所以，这也是你可以传递任意的数据或者方法给所有 resolvers 的方法。在这个例子中，你将会为 context 绑定 db 对象.

info 对象携带着 GraphQL 请求的信息（格式为 [query AST](https://medium.com/@cjoudrey/life-of-a-graphql-query-lexing-parsing-ca7c5045fad8)）。例如，在 query 的选择集中，它能够表示哪些字段是正在被请求的。

> 注：如果你想要更深入的学习 resolver 的参数，可以阅读下面两篇文章：

> [GraphQL Server Basics: The Schema](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e)

> [GraphQL Server Basics: Demystifying the info Argument in GraphQL Resolvers](https://www.prisma.io/blog/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a/)

现在你对 resolver 的参数有了一个基本的了解，我们来看看它们在 resolver 函数中将会如何被应用吧。

### 理解 feed resolver

feed resolver 的实现：

```js
feed: (root, args, context, info) => {
  return context.db.query.links({}, info)
},
```

它获取了 context 上的 db 对象。你将会看到，db 对象实际上就是一个来自  prisma-binding NPM 包的 Prisma binding 实例。

这个 Prisma binding 实例可以高效的吧 Prisma 数据库 schema 转化为你可以调用的 JS 函数。当调用这样的方法的时候，Prisma binding 实例将会自动为你组装一个 GraphQL 请求然后发送给 Prisma API。那么，你传递给 links 函数的两个参数都是什么呢？

第一个参数携带了你想要和 query 一同提交的参数。由于目前对于 link 的请求不需要任何参数，直接传递一个空对象即可。

第二个参数参数被 Prisma binding 实例用来生成发送给 Prisma API 的 query 的选择集。我们已经知道，这个 info 对象携带者当前请求的选择集。这里发生的事情就是，你把传入的请求委托给了 Prisma 引擎。info 对象需要被传递，这样 Prisma 就知道当前的请求在请求那些字段。

事实上，第二个字段也可以是一个字符串，它包含了请求的选择集：

```js
const selectionSet = `
{
  id
  description
  url
}
`
context.db.query.links({}, selectionSet)
```

它和如下发送给 Prisma API 的 GraphQL 请求可以对应：

```js
query {
  links {
    id
    description
    url
  }
}
```

### 理解 post resolver

post resolver 如下：

```js
post: (root, args, context, info) => {
  return context.db.mutation.createLink({
    data: {
      url: args.url,
      description: args.description,
    },
  }, info)
},
```

和 feed resolver 相似，你发起了一个和 context 绑定的 Prisma binding 实例方法。

之前说过的，实际上，Prisma binding 实例会将 Prisma 数据库 schema 转化为你可以用 js 调用的方法。调用这些方法将会发起相应的 query/mutation 给 Prisma API。

在这个例子中，你在 Prisma 的 GraphQL schema 发起了 createLink mutation。并将 resolvers 收到的数据通过 args 参数作为函数的参数传递给 Prisma API。

而 info 参数则和上面的例子一样，包含了 mutation 的选择集，它仍然可以用一个 string 来代替：

```js
const selectionSet = `
{
  id
}
`
context.db.mutation.createLink({
  data: {
    url: "www.prisma.io"
    description: "Prisma turns your database into a GraphQL API"
  }
}, selectionSet)
```

它可以和如下发送给 API 的 mutation 对应：

```js
mutation {
  createLink(data: {
    url: "www.prisma.io"
    description: "Prisma turns your database into a GraphQL API"
  }) {
    id
  }
```

总结一下： Prisma bindings 让你能够发起和 GraphQL schema 中定义的操作相对应的方法。这些方法和 GraphQL schema 中的操作同名，并且会根据参数的结构，返回相同结构的结果。

但是，你如何确定你的 resolver 能够访问到并记忆这些 Prisma binding 实例呢？

## 创建 Prisma binding 实例
