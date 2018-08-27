> * 原文地址：[Getting Started](https://www.howtographql.com/graphql-js/1-getting-started/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 开始吧！

在这部分章节中，你将搭建 GraphQL 服务并完成第一个 GraphQL 请求。最后，我们将会稍微讲解一下 GraphQL 的原理并学习 GraphQL schema。

## 创建项目

本篇教程将教会你如何从零开始搭建一个 GraphQL 服务，所以，你要做的第一件是就是创建一个目录来容纳服务所需的所有文件。

打开终端，导航到你选择放置文件夹的地址然后运行如下命令：

```js
mkdir hackernews-node
cd hackernews-node
yarn init -y
```

> 注：本篇教程使用 Yarn 来管理项目，如果你更偏向于 npm，你也可以用同等功能的 npm 命令。

上面的命令创建了一个名为 hackernews-node 的目录，完成初始化并创建 package.json 文件。package.json 是你正在搭建的 Node app 的配置文件。它列出了所有的依赖，以及其他 app 需要的配置选项（例如 scripts）。

## 创建一个最原始的 GraphQL 服务

有了项目目录文件，你就可以继续了 —— 创建 GraphQL 服务的入口。这是一个名为 index.js 的文件，位置在 src 目录下。

在终端，创建 src 目录然后创建空文件 index.js：

```js
mkdir src
touch src/index.js
```

执行 hackernews-node 目录下的 node src/index.js 就可以启动 app。但是目前什么也不会发生，因为 index.js 是一个空文件 ¯\_(ツ)_/¯（皮）

现在我们来开始建造这个 GraphQL 服务吧！要做的第一件事就是 —— 给项目添加一个依赖。

在终端执行：

```js
yarn add graphql-yoga
```

graphql-yoga 是一个全功能的 GraphQL 服务。它基于 Express.js 和一些其他的库，来帮助你搭建生产就绪的 GraphQL 服务。

下面是功能列表：

* GraphQL 规格兼容
* 支持文件上传
* GraphQL subscriptions 实时功能
* 支持 TypeScript 类型
* 支持 GraphQL Playground 的开箱即用
* 可通过 Express 中间件扩展
* 在 GraphQL schema 中可处理自定义指令
* 查询性能追踪
* 可接受两种 content-type：application/json 和 application/graphql
* 可以在各种环境下运行：可以通过 now, up, AWS Lambda, Heroku 等等部署

完美，现在来写一写代码吧 🙋

打开 src/index.js 然后输入如下代码：

```js
const { GraphQLServer } = require('graphql-yoga')

// 1
const typeDefs = `
type Query {
  info: String!
}
`

// 2
const resolvers = {
  Query: {
    info: () => `This is the API of a Hackernews Clone`
  }
}

// 3
const server = new GraphQLServer({
  typeDefs,
  resolvers,
})
server.start(() => console.log(`Server is running on http://localhost:4000`))
```

好，下面让我们了解下这段代码的功能，注解的序号和代码中注释的数字对应：

1. 常量 typeDefs 定义了 GraphQL schema（稍后详细解释这个）。这里就定义了一个简单的 Query 类型，并且它仅有一个属性：info。这个属性对应一个类型 String!。类型定义中的声明符号 ! 表示这个字段不能是 null。

2. resolvers 对象是 GraphQL schema 的实现。注意到，它的结构和：typeDefs: Query.info 内部类型定义的结构一致。

3. 最后，schema 和 resolvers 被捆绑起来然后传入 GraphQLServer，它是从 graphql-yoga 导入的。这个操作告知了服务什么样的 API 操作可以接受以及它们应该如何被处理。

现在，来测试一下这个 GraphQL 服务吧！

## 测试 GraphQL 服务

在项目的根目录下，运行如下命令：

```
node src/index.js
```

正如终端输出所示，服务现在运行在 http://localhost:4000。为了测试服务的 API，打开浏览器然后导航到这个地址。

你看到的是一个 GraphQL Playground，一个强大的 GraphQL IDE，可以让你以一种交互的方式测试 API 能力。

点击右侧的 SCHEMA 按钮，你可以看到 API 文档。这个文档是根据你定义的 schema 自动生成的，能够显示 schema 所有的 API 操作和数据类型。

下面让我们来发送第一个 GraphQL 请求。在左侧的编辑面板上输入如下代码：

```js
query {
  info
}
```

然后点击位于中间位置的 Play 按钮来将这个请求发送给服务（快捷键 CMD+ENTER）。

...运作中

恭喜你，现在你已经成功的发送并测试了你的第一个 GraphQL 请求 🎉

还记得之前我们说过：info: String!，这样的声明符号表示这个字段绝不能是 null。现在你已经成功实现了 resolver，知道如何控制字段的值了，那么如果你在 resolver 的实现时偏偏就返回 null 的话，会发生什么呢？我们来试一下。

更新 index.js 中 resolver 定义的代码：

```js
const resolvers = {
  Query: {
    info: () => null,
  }
}
```

为了测试结果，你需要重启服务：首先用 CTRL+C 来停止服务，然后运行 node src/index.js 重启。

现在，重新发送请求，这次，它返回了一个错误：Error: Cannot return null for non-nullable field Query.info。

之所以会这样，是因为底层的 graphql-js 确保了 resolver 中的返回类型和 GraphQL schema 的类型定义一致。换句话说，它能防止你犯一些低级错误。

这实际上是 GraphQL 的核心优势质疑：它强制 API 的行为和定义的 schema 一致。这样，所有看到 GraphQL schema 的人都能百分百的确定 API 的运行模式以及它返回的数据结构。

## GraphQL schema 简介

GraphQL schema 是所有 GraphQL API 的核心，我们来快速的介绍一下。

> 注：本篇教程就简单介绍一下 schema 的浅层知识。如果你想要深入了解 schema 和它在 API 中的角色，看看[这篇文章](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e)吧.

GraphQL schemas 的写作语法是 GraphQL Schema Definition Language（SDL）。SDL 是一个类型系统，允许定义数据结构（正如其他强类型语言 Java, TypeScript, Swift, Go 一样）。

这对 GraphQL 服务的 API 定义有什么帮助呢？所有的 GraphQL schema 都有三个特别的 root 类型：Query, Mutation 和 Subscription。root 类型分别负责三种 GraphQL 提供的操作类型：queries，mutations 和 subscriptions。root 类型下的字段被称为 root field，他们定义了可用的 API 操作。

例如，刚才我们使用的那个最简单的 GraphQL schema：

```js
type Query {
  info: String!
}
```

这个 schema 仅有一个 root field，名字是 info。当发送 queries, mutations 或者 subscriptions 给 GraphQL API 的时候，必须要以一个 root field 来开始。这里我们仅有一个 root field，所以 API 就只接受一种 query。

我们来看一个稍微复杂一点的例子：

```js
type Query {
  users: [User!]!
  user(id: ID!): User
}

type Mutation {
  createUser(name: String!): User!
}

type User {
  id: ID!
  name: String!
}
```

这个例子中，我们有了三个 root fields：Query 下有 users 和 user，Mutation 下有 createUser。另外还定义了 User 类型，否则 schema 的类型定义将不完整。（因为 Query 和 Mutation 中都用到了 User 类型）

从这个 schema 的定义中，可以派生出什么样的 API 操作呢？我们知道所有的 API 操作都必须以 root field 开始。但是我们还没有学过当 root field 本身的类型是另一个引用数据类型的时候，API 操作应该是什么样子。在这个例子中，root field 的类型分别是 [User!]!, User 和 User!。在前面的 info 的例子中，root field 的类型是 String，这是一个 [scalar type](http://graphql.org/learn/schema/#scalar-types)。

当 root field 是一个引用类型，你需要用这个引用类型的字段来继续扩展 query（或者 mutation/subscription）。这些扩展的部分称为：selection set。

如下这些是实现了上述 schema 的 API 可以接受的操作：

```js
// Query for all users
query {
  users {
    id
    name
  }
}

// Query a single user by their id
query {
  user(id: "user-1") {
    id
    name
  }
}

// Create a new user
mutation {
  createUser(name: "Bob") {
    id
    name
  }
}
```

有几点需要注意的是：

* 在这个例子中，我们对需要返回的 User 类型请求了 id 和 name。实际上，我们可以省略 id, name 的其中一个（也就是仅请求 id 或 name）。注意，当请求一个引用类型（object type）的数据时，需要至少请求 selection set 中的一个字段。

* 对于 selection set 中的字段，root field 的类型是 required（!） 或者 list 都可以。在上面这个 schema 的例子中，三个 root field 都有不同的 User 类型的 type modifiers（也就是 required 和 list 的不同组合）：

  * 对于 users 字段，返回值是 [User!]!，意味着它返回的是一个 User 类型元素的 list 列表（该字段本身不能为空）。列表中也不能包含 null 元素。所以，你总能对数据接收方保证，该字段下它们不会收到 null，并且列表中也不会包含空元素。

  * 对于 user(id: ID!) 字段，返回类型是 User 意味着可能是 null（没写 ! 所以可能为 null）或者是一个 User 类型的对象。

  * 对于 createUser(name: String!) 字段，返回类型是 User! 意味着必须返回 User 类型对象。

好了～理论知识够多了，让我们继续写写代码吧！（续下篇）

[self Proofreading +1]