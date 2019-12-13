> * 原文地址：[Realtime Updates with GraphQL Subscriptions](https://www.howtographql.com/react-apollo/8-subscriptions/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 使用订阅（subscription）实现实时更新

这一章我们将学习如何使用 GraphQL 订阅实现应用的实时更新。

## GraphQL 订阅是什么？

GraphQL 订阅允许服务器在某个特定的事件发生的时候，将信息推送给客户端。订阅通常是通过 [WebSockets](https://en.wikipedia.org/wiki/WebSocket) 实现的，此时服务器会和客户端建立稳定的连接。这意味着，当使用订阅时，会打破我们之前看到的那种客户端与 API 之间的请求-应答循环。客户端通过指定某个它感兴趣的事件，就可以和服务端保持连接。每当该指定的事件发生，服务端就会使用这个连接把信息推送给客户端。

## 使用 Apollo 实现订阅

首先需要用想要订阅的接口信息配置 ApolloClient。我们通过添加一个新的 Apollo 中间件 ApolloLink 来实现。这次我们将会使用 apollo-link-ws 包中的 WebSocketLink 函数。

首先将 apollo-link-ws 引入到项目中。导航到根目录，并在控制台中输入如下命令：

```sh
yarn add apollo-link-ws
yarn add subscriptions-transport-ws
```

下面使用订阅服务的信息配置 ApolloClient：

在 index.js 中添加如下代码：

```JavaScript
import { split } from 'apollo-link'
import { WebSocketLink } from 'apollo-link-ws'
import { getMainDefinition } from 'apollo-utilities'
```

注意：上面这段代码中，我们从 'apollo-link' 中引入了 split 函数。

下面，创建一个 WebSocketLink 实例，它代表了 WebSocket 连接。使用 split 方法正确处理请求，并更新调用 ApolloClient 构造函数的代码为如下所示：

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
  authLink.concat(httpLink)
)

const client = new ApolloClient({
  link,
  cache: new InMemoryCache()
})
```

如此就创建好了 WebSocketLink 实例，并配置了订阅的接口。这个例子中，订阅接口和 HTTP 接口很像，除了订阅功能使用 ws 协议，而不是 http 协议。我们注意到，这里也使用了从 localStorage 中获取的用户 token 授权 websocket 连接。

split 函数用来将请求导航至特定的中间件。它需要三个参数，第一个是测试函数，它将返回一个布尔值。另外两个参数都是 ApolloLink 类型。如果测试函数返回了 true，那么请求将被发送到第二个参数对应的链接；如果是 false，就使用第三个。

这个例子中，测试函数将会检验请求是属于订阅请求。如果是，请求就会使用 wsLink，否则就用原来的 authLink.concat(httpLink) 中间件。

![graphqlpic8](../imgs/graphqlpic8.png)

## 订阅新建事件

为了让应用能够在新的新闻链接被创建的时候实时更新，我们需要订阅 Link 类型发生变化的事件。如果使用了 Prisma，通常会有三种事件可以订阅：

* 创建了新的 Link 类型实例（即用户创建了一个新的新闻链接）

* 更新已经存在的 Link 类型实例

* 删除已经存在的 Link 类型实例

我们将会在 LinkList 组件内实现订阅新建事件的功能，因为这里可以展示出所有的新闻链接。

更新 LinkList.js 的代码：

```JavaScript
class LinkList extends Component {
  _updateCacheAfterVote = (store, createVote, linkId) => {
    const data = store.readQuery({ query: FEED_QUERY })
  
    const votedLink = data.feed.links.find(link => link.id === linkId)
    votedLink.votes = createVote.link.votes
  
    store.writeQuery({ query: FEED_QUERY, data })
  }

  _subscribeToNewLinks = async () => {
    // 我们稍后来实现这部分代码
  }

  render() {
    return (
      <Query query={FEED_QUERY}>
        {({ loading, error, data, subscribeToMore }) => {
          if (loading) return <div>Fetching</div>
          if (error) return <div>Error</div>

          this._subscribeToNewLinks(subscribeToMore)
    
          const linksToRender = data.feed.links
    
          return (
            <div>
              {linksToRender.map((link, index) => (
                <Link
                  key={link.id}
                  link={link}
                  index={index}
                  updateStoreAfterVote={this._updateCacheAfterVote}
                />
              ))}
            </div>
          )
        }}
      </Query>
    )
  }
}
```

代码解析：我们依旧使用 `<Query />` 组件，但是在它的 render prop 函数中，加入了新的参数 subscribeToMore。调用 _subscribeToNewLinks 函数并传入 subscribeToMore 参数即可完成组件对事件的订阅。这个函数内会建立与订阅服务的 ws 连接。

_subscribeToNewLinks 函数的实现如下：

```js
_subscribeToNewLinks = subscribeToMore => {
  subscribeToMore({
    document: NEW_LINKS_SUBSCRIPTION,
    updateQuery: (prev, { subscriptionData }) => {
      if (!subscriptionData.data) return prev
      const newLink = subscriptionData.data.newLink
      const exists = prev.feed.links.find(({ id }) => id === newLink.id);
      if (exists) return prev;

      return Object.assign({}, prev, {
        feed: {
          links: [newLink, ...prev.feed.links],
          count: prev.feed.links.length + 1,
          __typename: prev.feed.__typename
        }
      })
    }
  })
}
```

调用 subscribeToMore 需要两个参数：

1. document：代表了订阅请求本身。在这个例子中，订阅服务将在新建新闻链接的时候触发。

2. updateQuery：和 update 类似，这个函数允许你定义 store 在收到服务端的订阅推送后，应该如何更新信息。事实上，它和 Redux reducer 遵循同样的规则：它接受两个参数，store 中存储的旧状态，以及服务端推送的订阅信息。接下来你可以决定如何将订阅信息合并到现有状态中，然后返回更新的数据。这里的 updateQuery 函数中所实现的就是：从 subscriptionData 中获取已新建的新闻链接的信息，将它合并到现有列表中，然后返回合并后的结果。

最后，添加 NEW_LINKS_SUBSCRIPTION 相关的代码：

```js
const NEW_LINKS_SUBSCRIPTION = gql`
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
`
```

棒极了！现在你可以测试这个应用：打开两个窗口，第一个窗口访问 http://localhost:3000/，第二个窗口使用 playground 来发起一个可以新建新闻链接的 mutation 请求。此时发送请求后，应用界面将会实时更新。

> 注：如果运行报错的话，需要手动安装：yarn add subscriptions-transport-ws。

## 订阅点赞事件

和订阅新建事件很相似，详情可见 [LinkList.js](https://github.com/howtographql/react-apollo/blob/master/src/components/LinkList.js) 代码。
