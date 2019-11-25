> * 原文地址：[Server](https://www.howtographql.com/advanced/1-server/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# GraphQL 工具和生态圈

你可能已经意识到了，目前 GraphQL 的生态圈正在以高速发展。原因之一就是，GraphQL 为我们研发工具提供了很大的便捷。在这一章里，我们将会知道这一现象的原因，并向大家介绍一些生态圈中已有的很棒的工具。

如果你已经对 GraphQL 基础比较熟悉了，那么你或许知道 GraphQL 的类型系统是如何做到让我们可以快速定义 API 接口的。它不仅可以让开发者清晰的定义出 API 的功能，还可以使用 schema 校对传入的请求。

GraphQL 的优秀之处在于，这些功能不仅可以应用于服务端。GraphQL 也支持客户端向服务端请求 schema 信息。这就称为 introspection（内省）。

## introspection

schema 的设计者一定清楚 schema 是什么样的，但是客户端如何能通过 GraphQL API 发现那些信息是可以获取的呢？我们可以向 GraphQL 请求 __schema 这一元字段（meta-field），根据 GraphQL 规范，它在 query 的根类型上一定是可访问的。

```graphql
query {
  __schema {
    types {
      name
    }
  }
}
```

例如这个 schema 定义：

```graphql
type Query {
  author(id: ID!): Author
}

type Author {
  posts: [Post!]!
}

type Post {
  title: String!
}
```

如果我们发送上面那样的 introspection query，那么将会收到下面这样的结果：

```graphql
{
  "data": {
    "__schema": {
      "types": [
        {
          "name": "Query"
        },
        {
          "name": "Author"
        },
        {
          "name": "Post"
        },
        {
          "name": "ID"
        },
        {
          "name": "String"
        },
        {
          "name": "__Schema"
        },
        {
          "name": "__Type"
        },
        {
          "name": "__TypeKind"
        },
        {
          "name": "__Field"
        },
        {
          "name": "__InputValue"
        },
        {
          "name": "__EnumValue"
        },
        {
          "name": "__Directive"
        },
        {
          "name": "__DirectiveLocation"
        }
      ]
    }
  }
}
```

如你所见，我们请求到了 schema 中的所有类型。我们同时获取到了自定义的 object 和 scalar 类型，我们甚至还可以获取 introspection 类型！

在 introspection 类型中，还有很多字段是可以请求到的。这是另一个例子：

```graphql
{
  __type(name: "Author") {
    name
    description
  }
}
```

在这个例子中，我们请求了 __type 元字段以及它的 name 和 description。如下是这个请求的结果：

```graphql
{
  "data": {
    "__type": {
      "name": "Author",
      "description": "The author of a post.",
    }
  }
}
```

如你所见，introspection 是 GraphQL 的一个非常强大的特性，我们刚刚看到的只是冰山一角而已。通过 specification 我们可以知道更多关于 introspection schema 中有哪些可用的字段和类型的信息。

GraphQL 生态圈中很多工具都是利用 introspection 系统提供了非常棒的功能。例如文档浏览器，自动填充功能，代码生成器，一切皆有可能！其中一个最有用的称为 GraphiQL，当你构建和使用需要极大依赖 introspection 的 GraphQL API 的时候，你将会用得上它。

## GraphQL 练习场

[GraphQL Playground](https://github.com/prisma-labs/graphql-playground) 是一个非常强大的 “GraphQL IDE”，可用于与 GraphQL API 的交互测试。可用于编辑 GraphQL 的 query、mutation 和 subscription，可以完成代码的自动填充和校验，同时支持快速查看 schema 结构（这利用了 introspection）。它还可以显示请求历史，并让开发者可以并排处理多个 GraphQL API。同时它已经与 graphql-config 无缝集成。

它同时也是开发者的利器。例如，它能让你调试 GraphQL 服务或尝试发送请求，而无需重复编写 GraphQL query。
