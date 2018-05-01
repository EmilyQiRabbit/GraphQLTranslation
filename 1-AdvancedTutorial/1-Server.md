> * 原文地址：[Server](https://www.howtographql.com/advanced/1-server/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 服务端

GraphQL 经常被认为是前端开发的 API 技术，它让客户端获取数据的方式大大改进。但是这个 API 本身当然也可以应用于服务端，并且一样能带来很多优势，因为 GraphQL 能够让服务端开发者能够专注于描述数据，而不是实现和优化特定的接口。

## GraphQL 执行器

GraphQL 不仅指定了一个描述 schema 的方法，也不仅是一种获取数据的 query 语言，它实际是一个执行算法，将 query 转化为最终的结果。这个算法的核心其实很简单：query 的每个 field 被逐个转化，对每个 field 字段执行对应的 resolver。假设，我们有如下这样的 schema：

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

基于这个 schema，下面这个 query 就可以发送给服务端：

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

可以看出，每个 field 字段都可以和一个 type 联系：

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

现在我们可以很容易就看出服务的 resolvers 为每个 field 的运行。执行从 query 的 type 开始，并且宽度优先。也就是先运行 Query.author 的 resolver，当获取到结果并将结果传递给它的子元素 -- Author.posts 的 resolver。在这个层，结果是一个列表，所以在这个情境中，执行算法每一次只对一个项目执行。执行过程如下所示：

```JavaScript
Query.author(root, { id: 'abc' }, context) -> author
Author.posts(author, null, context) -> posts
for each post in posts
  Post.title(post, null, context) -> title
  Post.content(post, null, context) -> content
```

最后，执行算法将所有的东西放在一起然后整理好格式返回给客户端。

需要提醒一下的是，GraphQL 服务将会提供默认的 resolver，所以你不用为每个 field 专门指定 resolver。

关于 GraphQL 执行器的深度阅读可以参见[这篇博客](https://dev-blog.apollodata.com/graphql-explained-5844742f195e)

## 批量执行 resolver

你可能会觉得上面的执行策略很傻很天真。例如，如果你有一个从后台 API 或者数据库获取数据的 resolver，那么这个后台接口可能会在一次 query 中被执行多次。比如我们想获取几篇博文的作者：

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

如果这是同一个博客的博文，那么它们可能有同一个作者。所以如果我们需要为每个作者都发起一次 API 请求，我们可能会对同一条信息发起了多个请求：

```JavaScript
fetch('/authors/1')
fetch('/authors/2')
fetch('/authors/1')
fetch('/authors/2')
fetch('/authors/1')
fetch('/authors/2')
```

怎么解决这个问题？我们可以让 fetch 操作更聪明一些。我们可以把这个方法包裹在一个工具函数里，它将会等所有的 resolver 都发起后才统一执行，保证对每一个项目只发一次 fetch 请求：

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

还能做的更好么？当然，如果我们的 API 支持批请求，我们可以只发起一次请求，像这样：

```JavaScript
fetch('/authors?ids=1,2')
```

同样可以封装到一个 loader 函数中。

在 JS 中，上面这个策略可以用一个工具 [DataLoader](https://github.com/facebook/dataloader) 来实现，其他语言当然也有类似方法。