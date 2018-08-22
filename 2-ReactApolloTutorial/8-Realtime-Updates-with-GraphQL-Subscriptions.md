> * 原文地址：[Realtime Updates with GraphQL Subscriptions](https://www.howtographql.com/react-apollo/8-subscriptions/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 使用 GraphQL Subscriptions 完成实时更新

这一章是关于使用 GraphQL Subscriptions 来将实时更新的功能引入我们的应用。

## 什么是 GraphQL Subscriptions？

Subscriptions 也就是订阅，它是 GraphQL 的一项功能，允许服务器在某个特定的事件（也就是订阅的事件）发生的时候，将信息推送给客户端。Subscriptions 通常是用 WebSockets 实现的，这时，服务器和客户端建立了长连接。这意味着，当使用 Subscriptions 时，你就打破了之前那些组件使用的那种：请求-应答 的循环，客户端将和服务器保持连接并指定某个感兴趣的事件。每当这个事件发生，服务器将使用这个连接把信息推送给客户端。

## 使用 Apollo 完成 Subscriptions 订阅

如果使用 Apollo，你需要用 Subscriptions 配置 ApolloClient。要完成这项任务，需要为 Apollo 中间件链添加另外一个 ApolloLink。这一次是引了 apollo-link-ws 包中的 WebSocketLink。

首先将它作为依赖引入 app。

在目录 react-apollo 下的控制台输入如下命令：

```
yarn add apollo-link-ws
...
yarn add subscriptions-transport-ws
```

下面，配置 ApolloClient，让它“知道” subscription 服务的存在：

在 index.js 中添加如下代码：

```JavaScript
import { ApolloLink, split } from 'apollo-client-preset'
import { WebSocketLink } from 'apollo-link-ws'
import { getMainDefinition } from 'apollo-utilities'
```

下面，创建 WebSocketLink 实力代表 WebSocket 连接。并且使用 split 方法来对请求进行的“导航”（不同的请求将使用不同的中间件），并更新 ApolloClient 调用的方法：

```JavaScript
const wsLink = new WebSocketLink({
  uri: `ws://localhost:4000`,
  options: {
    reconnect: true,
    connectionParams: {
      authToken: localStorage.getItem(AUTH_TOKEN),
    }
  }
})

const link = split(
  ({ query }) => {
    const { kind, operation } = getMainDefinition(query)
    return kind === 'OperationDefinition' && operation === 'subscription'
  },
  wsLink,
  httpLinkWithAuthToken,
)

const client = new ApolloClient({
  link,
  cache: new InMemoryCache()
})
```

wsLink：如此便创建了 WebSocketLink 实例并且它绑定了 subscriptions。这个例子中，subscriptions 接口和 HTTP 接口其实很像，只是 subscriptions 使用的是 ws 协议而不是 http 协议。注意到，你也使用了用户的 token 信息来授权 websocket 连接。

split 函数是用来将一个 request 导航到特定的中间件。它需要三个参数，第一个是一个测试函数，返回一个布尔值。另外两个参数都是 ApolloLink 类型。如果测试函数返回了 true，那么请求将会使用第二个中间件，如果是 false，就使用第三个中间件。

这个例子中，测试函数将会检验请求是否是 subscription。如果是，就会使用 wsLink中间件，否则就用原来的 httpLinkWithAuthToken 中间件。

![graphqlpic8](../imgs/graphqlpic8.png)

## 对新的 links 发起订阅

为了让应用能够在新的 link 被创建的时候实时更新，我们需要对 Link 类型变化的事件发起订阅。当使用 Prisma 的时候，下面三个事件可以订阅：

* 创建了新的 Link

* 更新一个已经存在的 Link

* 删除一个已经存在的 Link

我们将会在 LinkList 组件上应用 subscription，因为这里是所有 link 展示的地方。

更新 LinkList.js 的代码：

```JavaScript
_subscribeToNewLinks = () => {
  this.props.feedQuery.subscribeToMore({
    document: gql`
      subscription {
        newLink {
          node {
            id
            url
            description
            createdAt
            postedBy {
              id
              name
            }
            votes {
              id
              user {
                id
              }
            }
          }
        }
      }
    `,
    updateQuery: (previous, { subscriptionData }) => {
      // ... you'll implement this in a bit
    }
  })
}
```

让我们来解析一下这段代码。首先引用 feedQuery（这个方法是在文件最后，用 graphql 包裹定义的）方法可以继续调用 subscribeToMore。这个方法可以打开 websocket 连接，去向服务器发起订阅。

subscribeToMore 方法需要两个参数：

1. document：代表了 subscription 请求本身。在这个例子中，subscription 将在 link 被创建的时候触发。

2. updateQuery：和 update 类似，这个函数允许你定义 store 在收到服务器的回应后如何更新信息。

下面我们就来实现 updateQuery。这个方法和 update 的工作原理稍有不同。其实它和 Redux reducer 的规则是一样的。它将前一个状态的 state 作为一个参数，另一个参数是服务器发回的订阅信息。现在你可以定义返回的订阅信息如何更新现有的 state，然后返回更新了的数据。

看看代码：

```JavaScript
updateQuery: (previous, { subscriptionData }) => {
  const newAllLinks = [subscriptionData.data.newLink.node, ...previous.feed.links]
  const result = {
    ...previous,
    feed: {
      links: newAllLinks
    },
  }
  return result
},
```

这里，把收到的 subscriptionData 中的新的 link 和已经有的 links 列表合并，然后返回结果。

最后，就是要确认组件订阅了事件，更新 LinkList.js 的代码，添加 componentDidMount 方法：

```JavaScript
componentDidMount() {
  this._subscribeToNewLinks()
}
```

componentDidMount 是 React 组件声明周期方法，将会在组件初始化后马上被调用。

> 运行报错的话，需要手动引用个包：yarn add subscriptions-transport-ws。

现在，你可以这样测试这个应用：打开两个窗口，第一个窗口访问 http://localhost:3000/，第二个窗口使用 playground 来发起 post mutation。当你发起了 post 请求，你就能看到 app 界面实时更新。

## 订阅新 votes

和订阅新 Link 很相似，详情可以直接去看 LinkList.js 的代码。

[self Proofreading +1]
