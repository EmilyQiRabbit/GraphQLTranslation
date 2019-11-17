> * 原文地址：[Authentication](https://www.howtographql.com/graphql-js/6-authentication/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 认证

本篇教程将会实现注册和登录，用户可以通过 GraphQL 服务认证了。

## 在数据 modal 中添加 User 类型

首先，你需要一个能够在数据库中表示用户数据的字段。解决方法就是在 datamodel 中添加 User 类型。

同时也需要在 User 和 Link 之间建立联系，来表示 Link 是哪个用户发起的。

打开文件：database/datamodel.graphql，添加如下代码：

```js
type Link {
  id: ID! @unique
  description: String!
  url: String!
  postedBy: User
}

type User {
  id: ID! @unique
  name: String!
  email: String! @unique
  password: String!
  links: [Link!]!
}
```

这里，在 Link 中添加了一个 postedBy 字段，指向 User 实例。User 类型则有一个 links 字段表示一个 Link 的数组。这是一个一对多的关系。

修改 data model 后，需要重新部署 Prisma 来应用这些改变。

在项目的根目录下运行：
```
prisma deploy
```

部署后，Prisma database schema 和 Prisma 服务的 API 都会被更新。这个 API 现在也会同时暴露出 User 类型的增删改查操作，以及根据指定的关系连接和断开 User 和 Link 的操作。

## 扩展应用 schema

还记得 schema 驱动开发的过程吗？它是从用你想要添加到 API 的新的操作、来扩展 schema 的定义开始的 —— 在这个例子中，就是注册和登录的 Mutation。

打开应用 schema：src/schema.graphql 文件，更新 Mutation：

```
type Mutation {
  post(url: String!, description: String!): Link!
  signup(email: String!, password: String!, name: String!): AuthPayload
  login(email: String!, password: String!): AuthPayload
}
```

下一步，在 src/schema.graphql 添加 AuthPayload 和 User 类型的定义：

```
type AuthPayload {
  token: String
  user: User
}

type User {
  id: ID!
  name: String!
  email: String!
  links: [Link!]!
}
```

注册和登录 mutation 行为其实非常类似。两者都返回注册（或登录）用户的信息和一个 token 信息，它可以用来认证后续的对 API 的请求。这个信息捆绑在 AuthPayload 类型中。

等一下？为什么你现在又一次定义了 User 类型？类型不能从 Prisma 数据库的 schema 中导入吗？答案是当然可以！

但是，在这个例子中，你重新定义 User 来在 App schema 中隐藏它的部分信息。比如，密码字段（正如你将会看到的：你将会用哈希版本来保存密码——所以，即使它在这里暴露出来，客户也无法直接查询它）。

## 实现 resolver 函数

在扩充了 schema 定义之后，我们就需要为它实现 resolver 函数了。在这之前，先完成代码重构，让它更加模块化。

将每个类型的 resolver 都提取出来放到单独的文件里。

首先，创建一个名为 resolvers 的文件，然后添加三个文件：Query.js、Mutation.js 和 AuthPayload.js。可以用以下命令完成：

```sh
mkdir src/resolvers
touch src/resolvers/Query.js
touch src/resolvers/Mutation.js
touch src/resolvers/AuthPayload.js
```

接下来，将 feed resolver 的实现挪到 Query.js 中。

在 Query.js 中添加如下函数的定义：

```js
function feed(parent, args, context, info) {
  return context.db.query.links({}, info)
}

module.exports = {
  feed,
}
```

很简洁明了，就只是将之前同样的函数移动到了各自专门的文件中。接下来是 Mutation resolver。

打开 Mutation.js 文件然后添加 login 和 signup resolver 函数（post resolver 稍后再添加）：

```js
async function signup(parent, args, context, info) {
  // 1
  const password = await bcrypt.hash(args.password, 10)
  // 2
  const user = await context.db.mutation.createUser({
    data: { ...args, password },
  }, `{ id }`)

  // 3
  const token = jwt.sign({ userId: user.id }, APP_SECRET)

  // 4
  return {
    token,
    user,
  }
}

async function login(parent, args, context, info) {
  // 1
  const user = await context.db.query.user({ where: { email: args.email } }, ` { id password } `)
  if (!user) {
    throw new Error('No such user found')
  }

  // 2
  const valid = await bcrypt.compare(args.password, user.password)
  if (!valid) {
    throw new Error('Invalid password')
  }

  const token = jwt.sign({ userId: user.id }, APP_SECRET)

  // 3
  return {
    token,
    user,
  }
}

module.exports = {
    signup,
    login,
    post,
}
```

下面，我们来根据标号逐一解释一下上面的代码，从 signup 开始。

1. signup Mutation 中，做的第一件事就是解码用户密码，所用工具是 bcryptjs 库。这个库稍后需要安装。

2. 下一步，使用 Prisma 绑定了的实例，在数据库保存新用户。注意到这里在 selection set 中 id 采用了硬编码。后续我们再深入讨论这一步。

3. 生成了一个带有 APP_SECRET 签名的 JWT。后续你将需要创建这个 APP_SECRET 并安装 jwt 库。

4. 最后，返回了 token 和创建了的 user。

现在来看 login mutation：

1. 这次不是创建用户，而是用 Prisma 的绑定实例来根据 login mutation 一同发送来的 email 信息查找已经存在的用户记录。如果没有发现同样的用户邮箱，就返回一个相关的错误。注意到，这次你将会同时在 selection set 中加入 id 和 password。密码当然是必须的，因为你需要用它和 login mutation 发来的密码做比对。

2. 下一步就是比较 mutation 提供的密码以及存储在数据库中的密码。如果两者不匹配，也需要返回错误。

3. 最后，依旧是返回 token 和 user。

两个 resolver 的实现都非常简单直接。最后一个还没搞清楚的问题就是 signup 的 selection set 硬编码中包含 id 字段。如果 signup mutation 还要求了更多的用户信息，会怎样呢？

## 添加 AuthPayload resolver

例如，考虑到如下这个在 GraphQL schema 中定义了的 mutation：

```js
mutation {
  login(
    email: "sarah@graph.cool"
    password: "graphql"
  ) {
    token
    user {
      id
      name
      links {
        url
        description
      }
    }
  }
}
```

这就是一个正常的 login mutation，但是需要服务器返回正在登录的用户的相关信息。那么 mutation 的这个 selection set（选择集合）应当如何解析呢？

目前的 resolver 实现只能返回用户 id，其他的信息均不返回。解决方案就是，实现一个附加的 AuthPayload resolver 函数，并取回 mutation 所要求的数据。

打开 resolver 目录下的 AuthPayload.js 文件并添加如下代码：

```js
function user(root, args, context, info) {
  return context.db.query.user({ where: { id: root.user.id } }, info)
}

module.exports = { user }
```

这里就是 login mutation 的 selection set（选择集合）如何解析并从数据库获取数据的。

> 注意：一开始就理解这里可能会有些困难。想要深入了解，可以参考这篇 [Github issue](https://github.com/prismagraphql/prisma/issues/1737) 或者阅读这篇[博客](https://www.prisma.io/blog/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a/)。

下面我们继续完成功能的实现。

首先，为项目添加必要的依赖：

```sh
yarn add jsonwebtoken bcryptjs
```

然后，创建一些可复用的工具方法：

目录 src 下创建文件 utiles.js

```sh
touch src/utils.js
```

然后粘贴如下代码：

```js
const jwt = require('jsonwebtoken')
const APP_SECRET = 'GraphQL-is-aw3some'

function getUserId(context) {
  const Authorization = context.request.get('Authorization')
  if (Authorization) {
    const token = Authorization.replace('Bearer ', '')
    const { userId } = jwt.verify(token, APP_SECRET)
    return userId
  }

  throw new Error('Not authenticated')
}

module.exports = {
  APP_SECRET,
  getUserId,
}
```

APP_SECRET 用来为分发给用户的 jsts 签名。它完全依赖于 prisma.yml 中定义的 secret。事实上，它和 Prisma 没有什么关系，如果你换一种方式来实现数据库层，APP_SECRET 也还是一样的使用方法。

getUserId 是一个辅助函数，你将会在需要认证的 resolver 中调用它。它首先会从 http 请求中取回包含用户 jwt 的 Authorization 头部信息。然后认证 jwt 并从中取得用户 id。注意到这一步可能不一定会成功执行，那么函数就会抛出一个错误。因此你也就能够利用它“保护”请求认证的 resolver。

最后，为 Mutation.js 添加 import：

```js
const bcrypt = require('bcryptjs')
const jwt = require('jsonwebtoken')
const { APP_SECRET, getUserId } = require('../utils')
```

## 为 post mutation 添加认证

在真正测试你的认证流之前，确保完成 schema/resolver 函数的构建。目前 post resolver 还没有写。

在 Mutation.js 中，添加 post resolver 的实现：

```js
function post(parent, args, context, info) {
  const userId = getUserId(context)
  return context.db.mutation.createLink(
    {
      data: {
        url: args.url,
        description: args.description,
        postedBy: { connect: { id: userId } },
      },
    },
    info,
  )
}
```

和前面的实现相比，上文的这个实现有两点不同：

1. 现在你用 getUserId 函数来从 jwt 存储中获取用户的 id，它被设置在了传入的 http 请求的认证头部里。因此，你就知道了是哪个用户创建了 Link。前文中提过，如果获取用户 id 失败，将会导致错误，那么函数就会在新建 Link 之前退出了。

2. 接下来你还需要用这个 userId 来链接当前这个用户创建的 Link。这个是通过 connect mutation 实现的。

棒极了！最后，你需要用信息 resolver 来实现 index.js 中的代码：

打开 index.js 然后在文件顶部引入包含 resolvers 的模块：

```js
const Query = require('./resolvers/Query')
const Mutation = require('./resolvers/Mutation')
const AuthPayload = require('./resolvers/AuthPayload')
```

然后更新 resolvers 对象：

```js
const resolvers = {
  Query,
  Mutation,
  AuthPayload
}
```

现在你可以测试一下这个认证流了！

## 测试

测试 signup mutation 的第一件事就是在数据库中新建一个 user。

> 如果你还没有这么做，ctrl+c 停止服务，然后运行 node src/index.js。之后通过在 GraphQL CLI 中运行 graphql playground 打开 playground。

注意到，只要你一直没有关闭页面，你就你可一重复利用你的 playground —— 重要的是你是否启动了服务，所以你做的变化能够被应用。

现在，发送如下的 mutation 来创建用户：

```js
mutation {
  signup(
    name: "Alice"
    email: "alice@graph.cool"
    password: "graphql"
  ) {
    token
    user {
      id
    }
  }
}
```

从服务端返回的应答中拷贝认证 token 然后打开另一个标签页。在这个新的标签页中，从左下角打开 HTTP HEADERS 面板然后填写好认证头部。

```js
{
  "Authorization": "Bearer __TOKEN__"
}
```

现在，无论你在这个面板发送 query 还是 mutation，它都会挟带这这个认证 token。

有了这个认证头部，发送如下的 post 请求到你的 GraphQL 服务：

```js
mutation {
  post(
    url: "www.graphql-europe.org"
    description: "Europe's biggest GraphQL conference"
  ) {
    id
  }
}
```

服务收到这个请求，就会触发 post resolver 函数，然后认证提供的 jwt。接下来，一个新的 Link 就被创建了。

为了确认一切正常工作，你可以发送如下 login mutation：

```js
mutation {
  login(
    email: "alice@graph.cool"
    password: "graphql"
  ) {
    token
    user {
      email
      links {
        url
        description
      }
    }
  }
}
```

[self Proofreading +1]
