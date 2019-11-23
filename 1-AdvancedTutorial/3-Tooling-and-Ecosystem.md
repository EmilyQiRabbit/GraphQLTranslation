> * 原文地址：[Server](https://www.howtographql.com/advanced/1-server/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# Tooling and Ecosystem

As you probably realized already, the GraphQL ecosystem is growing at an amazing speed right now. One of the reasons that this is happening is because GraphQL makes it really easy for us to develop great tools. In this section, we will see why this is the case, and a few amazing tools we already have in the ecosystem.

If you are familiar with GraphQL basics, you probably know how GraphQL’s Type System allows us to quickly define the surface area of our APIs. It allows developers to clearly define the capabilities of an API, but also to validate incoming queries against a schema.

An amazing thing with GraphQL is that these capabilities are not only known to the server. GraphQL allows clients to ask a server for information about its schema. GraphQL calls this introspection.

## Introspection

The designers of the schema already know what the schema looks like but how can clients discover what is accessible through a GraphQL API? We can ask GraphQL for this information by querying the __schema meta-field, which is always available on the root type of a Query per the spec.

```
query {
  __schema {
    types {
      name
    }
  }
}
```

Take this schema definition for example:

```
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

If we were to send the introspection query mentioned above, we would get the following result:

```
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

As you can see, we queried for all types on the schema. We get both the object types we defined and scalar types. We can even introspect the introspection types!

There’s much more than name available on introspection types. Here’s another example:

```
{
  __type(name: "Author") {
    name
    description
  }
}
```

In this example, we query a single type using the __type meta-field and we ask for its name and description. Here’s the result for this query:

```
{
  "data": {
    "__type": {
      "name": "Author",
      "description": "The author of a post.",
    }
  }
}
```

As you can see, introspection is an extremely powerful feature of GraphQL, and we’ve only scratched the surface. The specification goes into much more detail about what fields and types are available in the introspection schema.

A lot of tools available in the GraphQL ecosystem use the introspection system to provide amazing features. Think of documentation browsers, autocomplete, code generation, everything is possible! One of the most useful tools you will need as you build and use GraphQL APIs uses introspection heavily. It is called GraphiQL.

## GraphQL Playground

GraphQL Playground is powerful “GraphQL IDE” for interactively working with a GraphQL API. It features an editor for GraphQL queries, mutations and subscriptions, equipped with autocompletion and validation as well as a documentation explorer to quickly visualize the structure of a schema (powered by introspection). It also can display your query history or lets you work with multiple GraphQL APIs side-by-side. It also seamlessly integrates with graphql-config.

It is an incredibly powerful tool for development. It allows you to debug and try queries on a GraphQL server without having to write plain GraphQL queries over curl, for example.
