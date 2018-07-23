> * 原文地址：[Authentication](https://www.howtographql.com/graphql-js/6-authentication/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 认证

这个模块将会实现注册和登陆的功能，用户就可以被 GraphQL 服务认证了。

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

所以，注册和登录 mutation 行为其实非常类似。两者都返回注册（或登录）用户的信息和一个 token 信息，它可以用来认证后续的对 API 的请求。这个信息捆绑在 AuthPayload 类型中。

但是，等一下。为什么你现在又一次定义了 User 类型？类型不能从 Prisma 数据库的 schema 中导入吗？答案是当然可以！

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

很简洁明了。就只是将之前同样的函数移动到了专门、各自的文件中。接下来是 Mutation resolver。

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

1. 这次不是创建用户，而是用 Prisma 的绑定实例来根据 login mutation 一同发送来的 email 信息查找已经存在的用户记录。如果没有发现同样的用户邮箱，就返回一个相关的错误/注意到，这次你将会同时在 selection set 中加入 id 和 password。密码当然是必须的，因为你需要用它和 login mutation 发来的密码做比对。

2. 下一步就是比较 mutation 提供的密码以及存储在数据库中的密码。如果两者不匹配，也需要返回错误。

3. 最后，依旧是返回 token 和 user。

两个 resolver 的实现都非常简单直接。最后一个还没搞清楚的问题就是 signup 的 selection set 硬编码中包含 id 字段。如果 signup mutation 还要求了更多的用户信息，会怎样呢？

## 添加 AuthPayload resolver

待续.....















