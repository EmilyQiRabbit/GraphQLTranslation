> * 原文地址：[Authentication](https://www.howtographql.com/graphql-js/6-authentication/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 认证

本篇教程，我们将会学习如何实现注册和登录功能，这样用户就可以通过 GraphQL 服务完成认证。

## 添加 User 模型

首先，我们需要一个数据库用来代表用户数据的方式。即在 Prisma datamodel 中添加 User 类型。

同时也需要在 User 和 Link 之间添加关联，用来表示新闻链接 Link 是由用户 User 创建的。

打开文件：`database/datamodel.graphql`，添加如下代码：

```js
type Link {
  id: ID! @id
  description: String!
  url: String!
  postedBy: User
}

type User {
  id: ID! @id
  name: String!
  email: String! @unique
  password: String!
  links: [Link!]!
}
```

我们为 Link 类型添加了一个 postedBy 字段，它指向一个 User 实例。为 User 类型则添加了 links 字段，它是一个 Link 类型实例的数组。这是一个使用 SDL 表述的一对多关系。

修改 datamodel 文件后，需要重新部署 Prisma 来应用这些修改，同时也会修改数据库模式。

在项目的根目录 hackernews-node 下运行：

```sh
prisma deploy
```

此时 Prisma API 已被更新。同时，Prisma 客户端会在命令运行完成后自动生成，它暴露了新添加的 User 模型的所有增删改查方法。

## 扩展 GraphQL 模式

还记得模式驱动开发的过程吗？一切都是从使用新的 API 操作来扩展模式定义开始的 —— 在这个例子中，新的 API 就是注册/登录 mutation 请求。

打开应用模式定义文件 [src/schema.graphql](https://github.com/howtographql/graphql-js/blob/master/src/schema.graphql)，更新 Mutation 类型：

```graphql
type Mutation {
  post(url: String!, description: String!): Link!
  signup(email: String!, password: String!, name: String!): AuthPayload
  login(email: String!, password: String!): AuthPayload
}
```

下一步，在文件中添加 AuthPayload 和 User 类型的定义：

```graphql
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

注册和登录 mutation 请求的行为其实非常类似。两者都返回注册（登录）的用户信息和 token，该 token 可以用来认证后续的 API 请求。这个信息被打包在 AuthPayload 类型中。

不能忘了，我们还需要在 src/schema.graphql 文件的定义中，为 Link 模型添加 postedBy 字段，来完成 User 和 Link 关系的双向绑定：

```graphql
type Link {
  id: ID!
  description: String!
  url: String!
  postedBy: User
}
```

## 实现 resolver 函数

扩充模式定义后，我们还需要为它们实现相应的 resolver 函数。在此之前，我们先对代码进行重构，让代码更加模块化。

我们将每个类型的 resolver 都提取出来放到单独的文件里。

首先，创建一个名为 resolvers 的文件，然后添加三个文件：`Query.js`，`Mutation.js` 和 `AuthPayload.js`。可以用以下命令完成：

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

非常简洁明了，只是通过使用不同文件中的专用函数，来重新实现和之前一样的功能。接下来是 Mutation 的 resolver 函数。

### 添加认证相关的 resolver 函数

打开 Mutation.js 文件然后添加 login 和 signup 的 resolver 函数（post 的 resolver 函数我们稍后再添加）：

```js
async function signup(parent, args, context, info) {
  // 1
  const password = await bcrypt.hash(args.password, 10)
  // 2
  const user = await context.prisma.createUser({ ...args, password })

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
  const user = await context.prisma.user({ email: args.email })
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

下面，我们来根据标号逐一解释一下这段代码，从 signup 开始。

1. 在 signup mutation 中，我们首先使用 bcryptjs 解码用户密码。这个库稍后需要安装。

2. 下一步，使用 Prisma 客户端实例，在数据库保存新用户信息。

3. 接下来我们生成了一个带有 APP_SECRET 签名的 JWT。后续我们还需要创建这个 APP_SECRET 并安装 jwt 库。

4. 最后，使用 AuthPayload 类型对象返回 token 和已经创建了的 user 信息。

现在来看 login mutation：

1. 这次不是创建用户，而是使用 Prisma 客户端实例，根据发送来的 login mutation 的参数 email 信息查找已经存在的用户记录。如果没有找到该用户邮箱，就返回相应的错误。

2. 下一步，对照 mutation 提供的密码以及存储在数据库中的密码。如果两者不匹配，也需要返回错误。

3. 最后，依旧是返回 token 和 user 信息。

我们还要完成余下的步骤。

首先，为项目添加必要的依赖：

```sh
yarn add jsonwebtoken bcryptjs
```

添加工具函数：首先在根目录下执行如下代码。

```sh
touch src/utils.js
```

然后在该文件中添加代码：

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

APP_SECRET 将会用作 JWT 签名，然后我们将其发行给用户。

getUserId 是一个辅助函数，你可以在需要认证的 resolver 函数中调用它。它首先会从 context 中取出认证头部信息 Authorization。然后认证 jwt 并从中取得用户 id。注意到这一步不一定能执行成功，失败时函数就会抛出一个错误。这样你就能“保护”需要请求认证的 resolver 函数不会发生错误。

最后，为 Mutation.js 引入依赖：

```js
const bcrypt = require('bcryptjs')
const jwt = require('jsonwebtoken')
const { APP_SECRET, getUserId } = require('../utils')
```

到此为止，我们还有个小问题：需要使用 context 访问 request 对象。但是我们只为 context 绑定了 prisma 客户端实例，还并没有为 context 添加 request 对象。

我们只需要调整 index.js 中，GraphQLServer 实例化代码即可：

```js
const server = new GraphQLServer({
  typeDefs: './src/schema.graphql',
  resolvers,
  context: request => {
    return {
      ...request,
      prisma,
    }
  },
})
```

这里我们使用了一个函数返回 context 结果，而不是像之前那样直接使用一个对象。这种方法的优点是，我们可以将携带 GraphQL query 或 mutation 的 HTTP 请求也绑定到 context 上。这就让所有的 resolever 函数都能读取到认证信息头，并用来验证用户是否有权限发起当前的请求。

## 为 post mutation 添加认证

在测试认证流之前，我们还要确保完成 schema/resolver 函数的配置。目前 post resolver 还没有写。

在 Mutation.js 中，添加 post resolver 函数的实现：

```js
function post(parent, args, context, info) {
  const userId = getUserId(context)
  return context.prisma.createLink({
    url: args.url,
    description: args.description,
    postedBy: { connect: { id: userId } },
  })
}
```

和之前相比，上文的实现有两点不同：

1. 现在我们用 getUserId 函数获取用户的 id。ID 被存储于 HTTP 请求认证头部设置的 jwt 中。这样我们就知道了是哪个用户创建了新的 Link。前文中提过，如果获取用户 id 失败，函数将会报错。那么执行 createLink 之前函数就会返回。这样的话，GraphQL 应答将只会包含一个错误，告诉用户他没有通过认证。

2. 接下来使用这个 userId 来将这个被创建的 Link 与创建它的用户连接起来。连接是通过[嵌套对象语法 nested object writes](https://www.prisma.io/docs/-rsc6#nested-object-writes)完成的。

最后，我们还必须确保 User 和 Link 之间的关联能够正确的解析。

还记得之前我们省略了一部分 resolever 的实现吗：

```graphql
Link: {
  id: parent => parent.id,
  url: parent => parent.url,
  description: parent => parent.description,
}
```

但是现在，Link 的 postedBy 字段和 User 的 links 字段不能直接省略了。我们需要完成这两个字段的实现，因为 GraphQL 服务无法推断出这两个字段的数据应该如何取得。

在 resolvers/Link.js 中添加：

```js
function postedBy(parent, args, context) {
  return context.prisma.link({ id: parent.id }).postedBy()
}

module.exports = {
  postedBy,
}
```

注意，我们需要调用 prisma 客户端实例返回 Link 数据的 postedBy() 方法，因为这个 postedBy resolver 函数解析的是 schema.graphql 定义中 Link 类型的 postedBy 字段。

类似的，在 resolvers/User.js 中添加：

```js
function links(parent, args, context) {
  return context.prisma.user({ id: parent.id }).links()
}

module.exports = {
  links,
}
```

### 代码整理

我们需要修改 index.js 文件，引入相关模块，并修改 resolvers 的定义：

```js
const Query = require('./resolvers/Query')
const Mutation = require('./resolvers/Mutation')
const User = require('./resolvers/User')
const Link = require('./resolvers/Link')

//...

const resolvers = {
  Query,
  Mutation,
  User,
  Link
}
```

现在你可以测试这个认证流了！

## 测试

首先我们测试 signup mutation，成功后数据库中会新增一个 User 类型信息。

> 测试之前，使用 ctrl+c 停止服务，然后运行 `node src/index.js`。浏览器打开 http://localhost:4000，这个地址会运行 GraphQL Playground。

注意到，只要没有关闭页面，我们就可以重复使用这个 playground —— 但是还请务必重新启动服务器，这样我们刚才所做的代码更改才会生效，这一点很重要。

现在，发送如下的 mutation 来创建用户：

```graphql
mutation {
  signup(
    name: "Alice"
    email: "alice@prisma.io"
    password: "graphql"
  ) {
    token
    user {
      id
    }
  }
}
```

从服务端返回的应答中拷贝认证 token 然后打开 playground 另一个标签页。在这个新的标签页中，从左下角打开 HTTP HEADERS 面板然后填写好认证头部。

```json
{
  "Authorization": "Bearer __TOKEN__"
}
```

现在，无论你在这个面板发送任何 query 或 mutation 请求，它都会挟带这这个认证 token。

有了这个认证头部，发送如下 post 请求到 GraphQL 服务：

```graphql
mutation {
  post(
    url: "www.graphql-europe.org"
    description: "Europe's biggest GraphQL conference"
  ) {
    id
  }
}
```

服务收到这个请求，就会触发 post resolver 函数，然后认证请求提供的 jwt。接下来，一个新的新闻链接就被创建了，同时它会和刚才 signup 的用户相关联。

为了确认一切正常工作，你可以发送如下 login mutation：

```js
mutation {
  login(
    email: "alice@prisma.io"
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

服务端将会返回：

```json
{
  "data": {
    "login": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiJjanBzaHVsazJoM3lqMDk0NzZzd2JrOHVnIiwiaWF0IjoxNTQ1MDYyNTQyfQ.KjGZTxr1jyJH7HcT_0glRInBef37OKCTDl0tZzogekw",
      "user": {
        "email": "alice@prisma.io",
        "links": [
          {
            "url": "www.graphqlconf.org",
            "description": "An awesome GraphQL conference"
          }
        ]
      }
    }
  }
}
```
