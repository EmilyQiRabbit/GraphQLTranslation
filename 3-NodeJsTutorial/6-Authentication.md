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





















