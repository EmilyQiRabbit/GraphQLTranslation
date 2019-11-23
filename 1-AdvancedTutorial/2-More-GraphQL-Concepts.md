> * 原文地址：[Server](https://www.howtographql.com/advanced/1-server/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# More GraphQL Concepts

# Enhancing Reusability with Fragments

Fragments are a handy feature to help to improve the structure and reusability of your GraphQL code. A fragment is a collection of fields on a specific type.

Let’s assume we have the following type:

```JavaScript
type User {
  name: String!
  age: Int!
  email: String!
  street: String!
  zipcode: String!
  city: String!
}
```

Here, we could represent all the information that relates to the user’s physical address into a fragment:

```JavaScript
fragment addressDetails on User {
  name
  street
  zipcode
  city
}
```

Now, when writing a query to access the address information of a user, we can use the following syntax to refer to the fragment and save the work to actually spell out the four fields:

```JavaScript
{
  allUsers {
    ... addressDetails
  }
}
```

This query is equivalent to writing:

```JavaScript
{
  allUsers {
    name
    street
    zipcode
    city
  }
}
```

## Parameterizing Fields with Arguments

In GraphQL type definitions, each field can take zero or more arguments. Similar to arguments that are passed into functions in typed programming languages, each argument needs to have a name and a type. In GraphQL, it’s also possible to specify default values for arguments.

As an example, let’s consider a part of the schema that we saw in the beginning:

```JavaScript
type Query {
  allUsers: [User!]!
}

type User {
  name: String!
  age: Int!
}
```

We could now add an argument to the allUsers field that allows us to pass an argument to filter users and include only those above a certain age. We also specify a default value so that by default all users will be returned:

```JavaScript
type Query {
  allUsers(olderThan: Int = -1): [User!]!
}
```

This olderThan argument can now be passed into the query using the following syntax:

```JavaScript
{
  allUsers(olderThan: 30) {
    name
    age
  }
}
```

## Named Query Results with Aliases

One of GraphQL’s major strengths is that it lets you send multiple queries in a single request. However, since the response data is shaped after the structure of the fields being requested, you might run into naming issues when you’re sending multiple queries asking for the same fields:

```JavaScript
{
  User(id: "1") {
    name
  }
  User(id: "2") {
    name
  }
}
```

In fact, this will produce an error with a GraphQL server, since it’s the same field but different arguments. The only way to send a query like that would be to use aliases, i.e. specifying names for the query results:

```JavaScript
{
  first: User(id: "1") {
    name
  }
  second: User(id: "2") {
    name
  }
}
```

In the result, the server would now name each User object according to the specified alias:

```JavaScript
{
  "first": {
    "name": "Alice"
  },
  "second": {
    "name": "Sarah"
  }
}
```

## Advanced SDL

The SDL offers a couple of language features that weren’t discussed in the previous chapter. In the following, we’ll discuss those by practical examples.

### Object & Scalar Types

In GraphQL, there are two different kinds of types.

* Scalar types represent concrete units of data. The GraphQL spec has five predefined scalars: as String, Int, Float, Boolean, and ID.

* Object types have fields that express the properties of that type and are composable. Examples of object types are the User or Post types we saw in the previous section.

In every GraphQL schema, you can define your own scalar and object types. An often cited example for a custom scalar would be a Date type where the implementation needs to define how that type is validated, serialized, and deserialized.

### Enums

GraphQL allows you to define enumerations types (short enums), a language feature to express the semantics of a type that has a fixed set of values. We could thus define a type called Weekday to represent all the days of a week:

```JavaScript
enum Weekday {
  MONDAY
  TUESDAY
  WEDNESDAY
  THURSDAY
  FRIDAY
  SATURDAY
  SUNDAY
}
```

Note that technically enums are special kinds of scalar types.

### Interface

An interface can be used to describe a type in an abstract way. It allows you to specify a set of fields that any concrete type, which implements this interface, needs to have. In many GraphQL schemas, every type is required to have an id field. Using interfaces, this requirement can be expressed by defining an interface with this field and then making sure that all custom types implement it:

```JavaScript
interface Node {
  id: ID!
}

type User implements Node {
  id: ID!
  name: String!
  age: Int!
}
```

### Union Types

Union types can be used to express that a type should be either of a collection of other types. They are best understood by means of an example. Let’s consider the following types:

```JavaScript
type Adult {
  name: String!
  work: String!
}

type Child {
  name: String!
  school: String!
}
```

Now, we could define a Person type to be the union of Adult and Child:

```JavaScript
 union Person = Adult | Child
```

This brings up a different problem: In a GraphQL query where we ask to retrieve information about a Child but only have a Person type to work with, how do we know whether we can actually access this field?

The answer to this is called conditional fragments:

```JavaScript
{
  allPersons {
    name # works for `Adult` and `Child`
    ... on Child {
      school
    }
    ... on Adult {
       work
    }
  }
}
```
