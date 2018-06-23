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

在其他步骤之前，先做所有开发者都最喜欢的步骤：为项目添加一个新的依赖（???）

> 在项目的根目录下，运行如下命令

```js
yarn add prisma-binding
```

cool～现在你可以将 Prisma binding 实例和 context 绑定了，这样你的 resolvers 就可以获取到它。

> 在 index.js 中，像下面这样更新 GraphQLServer 的实例（注意，你需要用你自己的 Prisma 终端地址来替代下面的 endpoint 值）：

```js
const server = new GraphQLServer({
  typeDefs: './src/schema.graphql',
  resolvers,
  context: req => ({
    ...req,
    db: new Prisma({
      typeDefs: 'src/generated/prisma.graphql',
      endpoint: 'https://eu1.prisma.sh/public-graytracker-771/hackernews-node/dev',
      secret: 'mysecret123',
      debug: true,
    }),
  }),
})
```

这是诀窍：你用如下的信息实例化了 Prisma：

* typeDefs：它指向 Prisma 数据库 schema，它定义了 Prisma 的完整 CRUD GraphQL API。注意，此时你实际上还没有这个文件 - 我们稍后将会告诉你如何得到它。

* endpoint：这是 Prisma API 的端口地址。你需要用你自己的 Prisma 服务地址来替换掉它。

* secret：还记得之前说过的，所有向 Prisma API 发起的请求都需要在 HTTP 请求的 Authorization 头部添加一个 JWT 来认证吗？这个 JWT 需要被 prisma.yml 中的密码签名。虽然你没有做任何直接的对 Prisma API 的请求，但是这些请求都是通过 Prisma binding 实例来帮助你发送的，所以你需要告诉它密码这样它才能够生成 JWT 并和请求一起发送。

* debug：把 debug 标志位设置为 true 意味着所有 Prisma binding 实例发送的请求将会在控制台被打印出来。这是个很方便的方法，可以来观察实际发送给 Prisma 的 GraphQL 请求和修改。

> 最后，为了让这些都能工作，你需要在 index.js 中引入 Prisma。在文件的最开头加入如下的引用行：

```js
const { Prisma } = require('prisma-binding')
```

基本完成了，就剩下最后一步了，就是下载 Prisma 数据库 schema，它会在 Prisma 的构造函数中被引用。

## 下载 Prisma 数据库 schema

有多个方法可以访问到 GraphQL API 的 schema。在本教程，你将使用 GraphQL CLI 并结合 graphql-config。这也会带来你的日常工作流的改进。

> 首先，创建一个 .graphqlconfig 文件：

```
touch .graphqlconfig.yml
```

这个文件是 GraphQL CLI 主要的信息源。

在文件中添加如下的内容：

```
projects:
  app:
    schemaPath: src/schema.graphql
    extensions:
      endpoints:
        default: http://localhost:4000
  database:
    schemaPath: src/generated/prisma.graphql
    extensions:
      prisma: database/prisma.yml
```

这里都做了什么呢？你定义了两个 project。正如你猜测的那样，每个 project 都代表了一个 GraphQL API，应用层的和数据库层的。

对于每个 project，都定义了 schemaPath，这个就是定义了每个 API 的 GraphQL schema 地址。

对于 app 项目，你还要继续定义 endpoint，也就是 GraphQL server 开始运行时候的 URL。

而对于 database 项目，仅仅指出了 prisma.yml 的地址。事实上，提供了这个文件的地址也就提供了 Prisma service 的端口，因为所有有关的端口信息都在这个文件里。

这样的设置，有两个主要的优点：

* 你可以在 playground 中同时和两个 GraphQL API 交互

* 当使用 prisma deploy 部署服务的时候，Prisma CLI 会下载生成的 Prisma 数据库 schema 到本地特定的地址下。

Prisma CLI 也会使用 .graphqlconfig.yml 提供的信息。因此，你可以在根目录下运行 prisma 命令，而不是在 database 地址下了。

在项目的根目录下，运行 prisma deploy 来下载 Prisma 数据库 schema 到本地，具体地址是在 .graphqlconfig.yml 中定义了。

观察命令的输出，你可以看到它打印了如下一行：

```
Writing database schema to `src/generated/prisma.graphql`  1ms
```

看～这就是地址 src/generated/prisma.graphql 中的 Prisma 数据库 schema。

好啦，下面就是你可以开始测试服务的最后一步了！

> 打开 src/schema.graphql 然后删除 Link 类型。

哈？为什么？为什么要这么做？那么用于定义 feed 和 post 字段的 Link 类型的定义现在从哪里来呢？答案是，你将会导入它。

> 仍然在 src/schema.graphql 文件中，在顶部添加如下一句话：

```js
# import Link from "./generated/prisma.graphql"
```

这个导入语法还并不是官方 GraphQL 定义的，它来自 graphql-yoga 使用的 graphql-import 包，这个包被用于解析 .graphql 文件的依赖。

注意到在这个栗子中，如果不删除 Link 类型，其实也没有什么影响。但是，只定义 Link 类型一次、之后都是复用，这样会更加方便。否则每次你修改 Link 类型的时候，都要修改两个地方。

棒棒哒，现在你可以开始运行服务并测试 API 了。

## 在 Playground 中测试两个 GraphQL API 

[详细操作步骤](https://www.howtographql.com/graphql-js/5-connecting-server-and-database/)
