> * 原文地址：[Server](https://www.howtographql.com/advanced/1-server/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 服务端

由于因为 GraphQL 能让客户端获取数据的方式大大改进，所以它也经常被认为是专注于前端研发的 API 技术。但是就 API 本身而言，当然也可以应用于服务端。GraphQL 一样能为服务端带来很多优势，因为它能让服务端开发者专注于说明可用数据，而不是实现和优化特定的接口。

## GraphQL 的执行

GraphQL 不仅指定了描述 schema 的方法和从这些 schema 获取数据的 query 语言，它实际是一套可执行算法，用来规定如何将 query 转化为最终的数据结果。算法核心其实很简单：通过执行字段对应的 resolver，逐个转化 query 的每个字段。假设我们有如下的 schema：

```JavaScript
type Query {
  author(id: ID!): Author
}

type Author {
  posts: [Post]
}

type Post {
  title: String
  content: String
}
```

基于这个 schema，我们可以向服务端发送如下这样的 query：

```JavaScript
query {
  author(id: "abc") {
    posts {
      title
      content
    }
  }
}
```

可以看出，query 中每个字段都属于特定的 schema 中的类型：

```JavaScript
query: Query {
  author(id: "abc"): Author {
    posts: [Post] {
      title: String
      content: String
    }
  }
}
```

现在我们可以很容易得出结论，服务端会为每个字段运行相应的 resolver 函数。代码执行将会从 query 的类型开始，并且以宽度优先的方式。在这个例子中，会先运行 `Query.author` 的 resolver 函数。在获取到结果后，会将该结果传递给它的子元素 -- `Author.posts` 的 resolver 函数。下一层的结果是一个列表，所以在这个情况下，算法会逐个对列表中的每个项目执行。执行过程如下：

```JavaScript
Query.author(root, { id: 'abc' }, context) -> author
Author.posts(author, null, context) -> posts
for each post in posts
  Post.title(post, null, context) -> title
  Post.content(post, null, context) -> content
```

最后，算法会将所有的结果汇总，并整理为指定格式，返回给客户端。

需要注意，大部分 GraphQL 服务将会提供默认的 resolver 函数，所以并不用为每个字段都指定 resolver。例如在 GraphQL.js 中，当 resolver 的父级对象已经包含了命名正确的字段，那就不需要为这个子对象指定 resolver 函数了。

更多关于 GraphQL 执行的内容，可以参见 Apollo 博客中的[“GraphQL Explain”](https://dev-blog.apollodata.com/graphql-explained-5844742f195e)。

## 批量解析

你可能会觉得上面的执行策略非常原始。例如一个可从后台 API 或者数据库获取数据的 resolver 函数，那么这个后台的 API 可能会在一次请求中被执行多次。假设我们想获取几篇文章的作者，于是我们会发送这样的 query：

```JavaScript
query {
  posts {
    title
    author {
      name
      avatar
    }
  }
}
```

如果它们都是属于同一博客的文章，那么就可能多篇文章都是同一个作者。所以如果我们逐个为每个作者发起一次 API 请求，我们可能重复的请求同一个接口：

```JavaScript
fetch('/authors/1')
fetch('/authors/2')
fetch('/authors/1')
fetch('/authors/2')
fetch('/authors/1')
fetch('/authors/2')
```

怎么解决这个问题呢？答案是更加智能的执行 fetch 函数。我们可以把 fetch 方法包裹在一个工具函数里，这个工具函数将会等所有的 resolver 都返回后才会执行，保证对每个接口只发一次 fetch 请求：

```JavaScript
authorLoader = new AuthorLoader()

// Queue up a bunch of fetches
authorLoader.load(1);
authorLoader.load(2);
authorLoader.load(1);
authorLoader.load(2);

// Then, the loader only does the minimal amount of work
fetch('/authors/1');
fetch('/authors/2');
```

还能进一步优化吗？答案是肯定的。如果 API 能支持批量请求，我们可以只请求一次，像这样：

```JavaScript
fetch('/authors?ids=1,2')
```

它同样可以封装到上面的 loader 函数中。

在 JS 中，这个策略可以使用工具 [DataLoader](https://github.com/facebook/dataloader) 实现，当然其他语言也会存在类似工具。
