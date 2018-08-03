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

注意：

待续...

















