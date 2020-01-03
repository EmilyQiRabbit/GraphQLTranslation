> * 原文地址：[Filtering, Pagination & Sorting](https://www.howtographql.com/graphql-js/8-filtering-pagination-and-sorting/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 条件查找、分页和排序

本篇是这部分教程的最后一章，我们将完成实现 API 几个高级功能。我们目标是，通过提供条件查找和分页参数，让用户可以对 feed query 请求返回的 Link 类型元素作出限制。

## 条件查找

多亏 Prisma，开发者可以轻松实现条件查找功能。和前几篇教程相似，强大的 Prisma 引擎将会完成最困难的解析 query 的工作。我们需要做的仅是将服务端收到的 query 请求转发给它。

第一步是需要确定 API 想要暴露的条件查找类型。当前项目中，feed query 需要能接受一个表示条件的字符串。然后该请求仅需要返回在 url 或者 description 字段中包含这个字符串的 Link 元素。

为项目的模式的 feed query 添加一个 filter 字符串参数，表示条件:

```graphql
type Query {
  info: String!
  feed(filter: String): [Link!]!
}
```

下一步，更新 feed resolver 函数的实现，利用客户端提供的参数完成条件过滤功能。

打开 [`src/resolvers/Query.js`](https://github.com/howtographql/graphql-js/blob/master/src/resolvers/Query.js) 文件然后更新 feed resolver 函数：

```js
async function feed(parent, args, context, info) {
  const where = args.filter ? {
    OR: [
      { description_contains: args.filter },
      { url_contains: args.filter },
    ],
  } : {}

  const links = await context.prisma.links({
    where
  })
  return links
}
```

如果 query 请求没有提供 filter 参数，那么常量 where 就是一个空对象，这样 Prisma 引擎为 links 请求返回数据时，就不会使用任何条件对元素进行筛选。

而如果参数 args 中包含了 filter 属性，我们就需要构建 where 对象来描述过滤条件。这个 where 参数将会被 Prisma 用来筛选掉不符合条件的 Link 类型元素。

这就是条件查询功能全部代码了！可以开始测试这个允许条件查询的 API 啦 —— 我们使用这个简单的 query 请求：

```graphql
query {
  feed(filter:"QL") {
    id
  	description
    url
    postedBy {
      id
      name
    }
  }
}
```

## 分页

分页是一种比较棘手的 API 设计。一般有两种主流的解决方案：

* 限制 offset：通过提供所需元素的索引，来请求特定部分的数据列表（事实上，基本都是通过提供元素开始处索引（offset）以及元素数目（limit）来进行限制的）。

* 基于游标：这种分页模型相对先进。列表中每个元素都与唯一的 ID（即游标）关联。客户端分页功能需要提供开始元素的游标以及元素的数量。

上述两种方法 Prisma 都可支持。本篇教程我们学习第一种。

> 关于这两种分页方法的更多内容可以[参考这里](https://blog.apollographql.com/understanding-pagination-rest-graphql-and-relay-b10f835549e7)

数目限制和起始坐标在 Prisma API 中的名称并不是 limit 和 offset：

* limit 被称为 first，意味着你将会获取提供的开始坐标后的数个元素。注意，还有一个可选的 last 参数，对应的将会返回倒数数个元素。

* 起始坐标 offset 则称为 skip，表示跳过列表开头的数个元素。如果没有提供 skip 参数，那么它默认是 0。此时分页就会从列表的第一个元素（如果你使用了 last，那就是从倒数第一个元素）开始返回。

下面我们来为 feed query 请求添加 skip 和 first 参数：

打开 [schema.graphql](https://github.com/howtographql/graphql-js/blob/master/src/schema.graphql) 文件：

```graphql
type Query {
  info: String!
  feed(filter: String, skip: Int, first: Int): [Link!]!
}
```

然后打开 `src/resolvers/Query.js`，调整 feed resolver 函数的实现：

```js
async function feed(parent, args, context, info) {
  const where = args.filter ? {
    OR: [
      { description_contains: args.filter },
      { url_contains: args.filter },
    ],
  } : {}

  const links = await context.prisma.links({
    where,
    skip: args.skip,
    first: args.first
  })
  return links
}
```

修改完成。调用 prisma.links 时新增了两个参数，它们都可能在传入的 args 参数中。而那些复杂的工作，Prisma 都会帮我们完成 🙏。

我们可以用如下的 query 请求来测试分页接口，它将会返回列表中的第二个 Link：

```graphql
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

使用 Prisma 也可以实现返回按照特定规则排序的列表。例如，可以通过 Link 元素的 url 和 description 字段的字母顺序排序。对于我们的 Hacker News API，你可以将如何排序的决定权交给用户，并在 GraphQL 服务端 API 中加入所有 Prisma API 支持的排序选项。我们可以创建一个枚举类型，代表所有排序选项。

将下面代码添加到 schema.graphql：

```graphql
enum LinkOrderByInput {
  description_ASC
  description_DESC
  url_ASC
  url_DESC
  createdAt_ASC
  createdAt_DESC
}
```

它代表了所有 Link 类型元素不同的排序的方式。

打开 schema.graphql 文件，更新 feed query，为其添加一个 orderBy 参数：

```graphql
type Query {
  info: String!
  feed(filter: String, skip: Int, first: Int, orderBy: LinkOrderByInput): [Link!]!
}
```

resolver 函数的实现和分页很类似。

打开 `src/resolvers/Query.js` 文件，更新 feed resolver 函数的实现并将 orderBy 参数传递给 Prisma：

```js
async function feed(parent, args, context, info) {
  const where = args.filter ? {
    OR: [
      { description_contains: args.filter },
      { url_contains: args.filter },
    ],
  } : {}

  const links = await context.prisma.links({
    where,
    skip: args.skip,
    first: args.first,
    orderBy: args.orderBy
  })
  return links
}
```

完美！下面这个 query 请求将会根据创建时间返回 Link 类型元素：

```graphql
query {
  feed(orderBy: createdAt_ASC) {
    id
    description
    url
  }
}
```

## 获取 Link 元素总数量

最后，我们还需要为 Hacker News API 添加一个可以返回当前数据库中所有 Link 元素总数量的功能。首先我们需要稍微重构一下 feed query 并创建一个新的类型 Feed。

在应用模式文件 schema.graphql 中添加 Feed 类型。然后调整 feed query 的返回值类型：

```graphql
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
async function feed(parent, args, context) {
  const where = args.filter
    ? {
        OR: [
          { description_contains: args.filter },
          { url_contains: args.filter },
        ],
      }
    : {}

  const links = await context.prisma.links({
    where,
    skip: args.skip,
    first: args.first,
    orderBy: args.orderBy,
  })
  const count = await context.prisma
    .linksConnection({
      where,
    })
    .aggregate()
    .count()
  return {
    links,
    count,
  }
}
```

1. 使用参数中提供的条件过滤、排序和分页参数取回相应的 Link 类型元素。

2. 下一步，使用 linksConnection 请求 Prisma 客户端 API，取回数据库中 Link 类型元素的总数。

3. links 和 count 会被存储于 Feed 类型的对象中。

最后一步，在 GraphQLServer 实例化的时候引入这个新的 resolver。

可以使用这个 query 请求进行测试：

```graphql
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
