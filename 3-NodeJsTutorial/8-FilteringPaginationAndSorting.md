> * 原文地址：[Filtering, Pagination & Sorting](https://www.howtographql.com/graphql-js/8-filtering-pagination-and-sorting/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 过滤器，分页和排序

这是本部分的最后一篇教程，你将在这里完成 API 最后一部分的实现。目标是：通过提供过滤器和分页参数，允许用户限制 feed query 请求返回的 Link 元素的数量。

## 过滤器

Prisma 可以让开发者很轻松的实现过滤器功能。和前几篇教程相似，强大的 Prisma 引擎将会完成最困难的解析 query 的工作。你需要做的仅仅就是将 query 发送给它。

第一步是需要确定你的 API 想要暴露的过滤器类型。在当前项目中，feed query 将会接受一个过滤字符串。然后该请求仅需要返回 url 或者 description 包含这个字符串的 Link 元素。

为项目的 schema 的 feed query 添加 filter 字符串:

```js
type Query {
  info: String!
  feed(filter: String): [Link!]!
}
```

下一步，更新 feed resolver 的实现：添加客户端提供的新的参数（也就是 filter string）的应用。

打开 src/resolvers/Query.js 文件然后更新 feed resolver：

```js
function feed(parent, args, context, info) {
  const where = args.filter
    ? {
        OR: [
          { url_contains: args.filter },
          { description_contains: args.filter },
        ],
      }
    : {}

  return context.db.query.links({ where }, info)
}
```

如果 query 没有提供 filter 参数，那么 where 就是一个空对象，那么当 prisma 引擎为 query 返回数据的时候，也就没有任何过滤条件被应用了。

而如果传入的参数重包含了 filter，就需要构建一个 where 对象来描述两个过滤条件。这个 where 参数将会被 Prisma 用来过滤掉不符合条件的 link 元素。

注意：Prisma binding 对象把上面的函数转译成类似下面这样的 GraphQL 请求。这个 query 请求是 Yoga 服务发送给 Prisma API 并解析的：

```js
query ($filter: String) {
  links(where: {
    OR: [{ url_contains: $filter }, { description_contains: $filter }]
  }) {
    id
    url
    description
  }
}
```

> 更多相关过滤器的功能的内容可以去阅读[官方文档](https://www.prisma.io/docs/reference/prisma-api/queries-ahwee4zaey/#filtering-by-field)。另一个学习的方法就是用 Prisma API 提供的 Playground，阅读 schema 文档对 where 的说明，或者直接在 Playground 中进行测试。

这就是过滤器功能的全部代码了，可以测试过滤功能 API 了 - 一个可用来测试的例子：

```js
query {
  feed(filter: "Prisma") {
    id
    description
    url
  }
}
```

## 分页

在 API 设计中，分页是一个很 tricky 的话题（这个 tricky 真的是很不好翻译啊...只可意会不可言传...总之不是什么好词儿）。一般会有两种比较好的解决方案：

* 限制 offset：提供所需条目的指标，来请求特定部分的数据（事实上，基本都是提供数据开始的 index(offset) 以及数据的条数来限制数据）。

* 基于游标：这样的分野模型相对先进。列表中的每个元素都和一个唯一的 ID（cursor）关联。客户端的分页功能需要提供开始元素的 cursor 以及元素的数量。

Prisma 支持上述两种方法。在这篇教程中，我们将会学习第一种。

> 关于这两种分页方法的更多内容可以[参考这里](https://blog.apollographql.com/understanding-pagination-rest-graphql-and-relay-b10f835549e7)

条目限制（Limit）和起始坐标（offser）在 Prisma API 中的名称是不同的。

* limit 被称为 first，意味着你将会获取提供的开始坐标后的前 x 个元素。注意，还有一个可选的 last 参数，对应的将会返回倒数的 x 个元素。

* 起始坐标则称为 skip，因为你跳过（忽略）了列表开头的数个元素。如果客户端没有提供 skip，那么默认就是 0。那么分页就会从列表的第一个元素（或者倒数第一个）开始返回。

下面，打开应用的 schema，我们就来为 feed 请求添加 skip 和 first 参数：

```js
type Query {
  info: String!
  feed(filter: String, skip: Int, first: Int): [Link!]!
}
```

现在，修改 resolver 的实现：

打开 src/resolvers/Query.js，调整 feed resolver 的实现：

```js
function feed(parent, args, context, info) {
  const where = args.filter
    ? {
        OR: [
          { url_contains: args.filter },
          { description_contains: args.filter },
        ],
      }
    : {}

  return context.db.query.links(
    { where, skip: args.skip, first: args.first },
    info,
  )
}
```

好了，修改完成了。对 link 的请求现在新增了两个参数。其余的一些细节的复杂的工作，Prisma 都会帮我们完成。

下面你可以用如下的 query 来测试分页接口，它将会返回列表中的第二个 Link：

```js
query {
  feed(
    first: 1
    skip: 1
  ) {
    id
    description
    url
  }
}
```

## 排序

使用 Prisma，返回特定规则排序的列表也是可实现的。例如，可以通过 link 的 url 和 description 的字母顺序来排序。对于我们的 Hackernews API，你可以将决定权给用户，让他们决定如何排序，并提供所有 Yoga server 支持的排序方式。通过引入 Prisma database schema 的 LinkOrderByInput 你就可以完成这点：

```js
enum LinkOrderByInput {
  id_ASC
  id_DESC
  description_ASC
  description_DESC
  url_ASC
  url_DESC
  updatedAt_ASC
  updatedAt_DESC
  createdAt_ASC
  createdAt_DESC
}
```

它代表了 Link 元素排序的方式。

打开应用的 schema 然后从 Prisma database schema 引入 LinkOrderByInput：

```js
# import Link, LinkSubscriptionPayload, Vote, VoteSubscriptionPayload, LinkOrderByInput from "./generated/prisma.graphql"
```

然后，再次调整 feed query 并加入 orderBy 参数：

```js
type Query {
  info: String!
  feed(filter: String, skip: Int, first: Int, orderBy: LinkOrderByInput): [Link!]!
}
```

resolver 的实现和分页的实现很类似。

打开 src/resolvers/Query.js 文件并更新 feed resolver 函数的实现并且传递 orderBy 参数给 Prisma：

```js
function feed(parent, args, context, info) {
  const where = args.filter
    ? {
        OR: [
          { url_contains: args.filter },
          { description_contains: args.filter },
        ],
      }
    : {}

  return context.db.query.links(
    { where, skip: args.skip, first: args.first, orderBy: args.orderBy },
    info,
  )
}
```

超棒～下面这个 query 将会根据创建时间返回 link 元素：

```js
query {
  feed(orderBy: createdAt_ASC) {
    id
    description
    url
  }
}
```

## 返回 Link 元素的总数

最后，你需要为你的 API 添加一个：返回当前数据库中 Link 元素总数的功能。为了完成这个功能，你需要稍微重构一下 feed query 并创建一个新的类型 Feed。

在应用 schema 中添加 Feed 类型。然后调整 feed query 的返回值类型：

```js
type Query {
  info: String!
  feed(filter: String, skip: Int, first: Int, orderBy: LinkOrderByInput): Feed!
}

type Feed {
  links: [Link!]!
  count: Int!
}
```

然后，再次调整 feed resolver 函数：

```js
async function feed(parent, args, context, info) {
  const where = args.filter
    ? {
        OR: [
          { url_contains: args.filter },
          { description_contains: args.filter },
        ],
      }
    : {}

  // 1
  const queriedLinks = await context.db.query.links(
    { where, skip: args.skip, first: args.first, orderBy: args.orderBy },
    `{ id }`,
  )

  // 2
  const countSelectionSet = `
    {
      aggregate {
        count
      }
    }
  `
  const linksConnection = await context.db.query.linksConnection({}, countSelectionSet)

  // 3
  return {
    count: linksConnection.aggregate.count,
    linkIds: queriedLinks.map(link => link.id),
  }
}
```

1. 首先使用已经完成的 filter order 和 pagination 参数取回 Link 元素。不同的是，你现在为 query 硬编码了一个选择域 `{ id }` 并且只取回 Link 元素的 id 字段。事实上，如果你在这里试图传递 info，API 将会报错。（[这里](https://www.prisma.io/blog/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a/)解释了原因）

2. 下一步，使用 linksConnection 请求 Prisma database schema 来取回 存在于数据库中的 Link 元素总数。使用硬编码设定选择域并通过 aggregate 字段返回元素总数 count.

3. count 会被直接返回。Feed 类型指定的 links 此时还没有返回 - 当你依旧需要执行下一级的 resolver 的时候它才会返回。因为这一层级的 resolver 仅返回 LinkId，下一级的 resolver 解析才会返回 link 元素对象。

最后一步，实现 Feed 类型的 resolver。

在文件 src/resolvers 中新建 Feed.js：

```js
touch src/resolvers/Feed.js
```

添加如下代码：

```js
function links(parent, args, context, info) {
  return context.db.query.links({ where: { id_in: parent.linkIds } }, info)
}

module.exports = {
  links,
}
```

这里是 Feed 类型中的 links 字段真正被解析的地方。如你所见，上一级的 resolver 的返回值将会携带者 linkIds 参数，并传入本级 resolve。

最后，在 GraphQLServer 实例化的时候引入这个新的 resolver。

在 index.js 文件中，添加：

```js
const Feed = require('./resolvers/Feed')
...
const resolvers = {
  Query,
  Mutation,
  AuthPayload,
  Subscription,
  Feed
}
```

下面就可以发起测试了：

```js
query {
  feed {
    count
    links {
      id
      description
      url
    }
  }
}
```
