> * 原文地址：[More Mutations and Updating the Store](https://www.howtographql.com/react-apollo/6-more-mutations-and-updating-the-store/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 更多 mutation 请求实例，以及 store 的更新

下面我们来实现为新闻链接点赞的功能。已登录用户将可以为新闻链接点赞，票数最高的新闻链接将会被展示在一个独立的页面中！

## 准备 React 组件

第一步要做的依旧是准备 React 组件。

打开 [Link.js](https://github.com/howtographql/react-apollo/blob/master/src/components/Link.js) 文件，更新 render 方法并且记得引用需要的文件：

```JavaScript
import { AUTH_TOKEN } from '../constants'
import { timeDifferenceForDate } from '../utils'
...
render() {
  const authToken = localStorage.getItem(AUTH_TOKEN)
  return (
    <div className="flex mt2 items-start">
      <div className="flex items-center">
        <span className="gray">{this.props.index + 1}.</span>
        {authToken && (
          <div className="ml1 gray f11" onClick={() => this._voteForLink()}>
            ▲
          </div>
        )}
      </div>
      <div className="ml1">
        <div>
          {this.props.link.description} ({this.props.link.url})
        </div>
        <div className="f6 lh-copy gray">
          {this.props.link.votes.length} votes | by{' '}
          {this.props.link.postedBy
            ? this.props.link.postedBy.name
            : 'Unknown'}{' '}
          {timeDifferenceForDate(this.props.link.createdAt)}
        </div>
      </div>
    </div>
  )
}
```

这个 Link 组件可以展示出每条新闻链接获得的点赞数，以及创建它的用户的姓名。如果用户已登录，也会显示点赞按钮。如果这个新闻链接的作者未知，即它没有和任何用户相关联，用户名将会显示为 `Unknown`。

注意到，这里我们还使用了一个 timeDifferenceForDate 方法，为它传入的参数是链接创建的时间，这个方法将会吧时间戳转化为比较友好的展示方式，比如：三个小时前。

这个方法的代码在 [util.js](https://github.com/howtographql/react-apollo/blob/master/src/utils.js) 文件里：

```JavaScript
function timeDifference(current, previous) {
  const milliSecondsPerMinute = 60 * 1000
  const milliSecondsPerHour = milliSecondsPerMinute * 60
  const milliSecondsPerDay = milliSecondsPerHour * 24
  const milliSecondsPerMonth = milliSecondsPerDay * 30
  const milliSecondsPerYear = milliSecondsPerDay * 365

  const elapsed = current - previous

  if (elapsed < milliSecondsPerMinute / 3) {
    return 'just now'
  }

  if (elapsed < milliSecondsPerMinute) {
    return 'less than 1 min ago'
  } else if (elapsed < milliSecondsPerHour) {
    return Math.round(elapsed / milliSecondsPerMinute) + ' min ago'
  } else if (elapsed < milliSecondsPerDay) {
    return Math.round(elapsed / milliSecondsPerHour) + ' h ago'
  } else if (elapsed < milliSecondsPerMonth) {
    return Math.round(elapsed / milliSecondsPerDay) + ' days ago'
  } else if (elapsed < milliSecondsPerYear) {
    return Math.round(elapsed / milliSecondsPerMonth) + ' mo ago'
  } else {
    return Math.round(elapsed / milliSecondsPerYear) + ' years ago'
  }
}

export function timeDifferenceForDate(date) {
  const now = new Date().getTime()
  const updated = new Date(date).getTime()
  return timeDifference(now, updated)
}
```

最后，每个 Link 组件都需要显示出它在列表中的序列号，所以我们将 index 从 LinkList 组件传递到 Link 组件中。

打开 [LinkList.js](https://github.com/howtographql/react-apollo/blob/master/src/components/LinkList.js) 文件，更新 render 函数中 Link 组件的渲染代码，加上 link 的序列号。

```JavaScript
// <Query query={FEED_QUERY}>
// ...
return (
  <div>
    {linksToRender.map((link, index) => (
      <Link key={link.id} link={link} index={index} />
    ))}
  </div>
)
// ...
// </Query>
```

此时 votes 参数还没有加入到 query 请求中，需要我们继续修改代码。

修改 LinkList.js 文件中 FEED_QUERY 的定义为：

```JavaScript
const FEED_QUERY = gql`
  {
    feed {
      links {
        id
        createdAt
        url
        description
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

这个 query 的数据负载中包括了创建者的信息以及点赞信息。

组件准备就绪了，下面我们进入点赞功能的实现!

## 发起 mutation 请求

在 [Link.js](https://github.com/howtographql/react-apollo/blob/master/src/components/Link.js) 中添加如下定义 mutation 的代码：

```JavaScript
import { Mutation } from 'react-apollo'
import gql from 'graphql-tag'
...
const VOTE_MUTATION = gql`
  mutation VoteMutation($linkId: ID!) {
    vote(linkId: $linkId) {
      id
      link {
        votes {
          id
          user {
            id
          }
        }
      }
      user {
        id
      }
    }
  }
`
```

然后，修改类名为 flex items-center 的 dev 元素：

```js
import { Mutation } from 'react-apollo'
import gql from 'graphql-tag'

// ...

<div className="flex items-center">
  <span className="gray">{this.props.index + 1}.</span>
  {authToken && (
    <Mutation mutation={VOTE_MUTATION} variables={{ linkId: this.props.link.id }}>
      {voteMutation => (
        <div className="ml1 gray f11" onClick={voteMutation}>
          ▲
        </div>
      )}
    </Mutation>
  )}
</div>
```

这一步操作大家应该都很熟悉了。通过添加 `<Mutation />` 组件，存在于该组建内部函数的组件获得了调用 voteMutation 的能力。

运行 yarn start，点击点赞按钮，刷新页面就可以看到结果了。

下面我们来学如何让页面在点赞后自动更新。

## 更新本地缓存

Apollo 可以帮助你手动控制缓存内容。这个功能非常方便，尤其是应用于发起 mutation 之后。它能让你精确定义如何更新缓存。现在我们使用这个功能来确保用户点赞后，UI 界面能立即显示正确的赞数。

我们使用 Apollo 的 caching data 来实现这个功能。

更新 Link.js 文件的 `<Mutation />` 组件，添加一个 update 属性：

```JavaScript
<Mutation
  mutation={VOTE_MUTATION}
  variables={{ linkId: this.props.link.id }}
  update={(store, { data: { vote } }) =>
    this.props.updateStoreAfterVote(store, vote, this.props.link.id)
  }
>
  {voteMutation => (
    <div className="ml1 gray f11" onClick={voteMutation}>
      ▲
    </div>
  )}
</Mutation>
```

上面代码中，作为 `<Mutation />` 组件 update 属性的函数将会在服务器返回结果后被执行。它的参数是 mutation 的数据负载（data）以及当前缓存 store。你可以使用它们更新缓存状态。

注意，此时你已经解析了服务端的信息，并取到了 vote 字段的信息。

真正的更新操作将会在 Link 的父级元素 LinkList 里完成 —— 打开 LinkList.js 文件并添加如下代码：

```JavaScript
_updateCacheAfterVote = (store, createVote, linkId) => {
  const data = store.readQuery({ query: FEED_QUERY })

  const votedLink = data.feed.links.find(link => link.id === linkId)
  votedLink.votes = createVote.link.votes

  store.writeQuery({ query: FEED_QUERY, data })
}
```

代码解释：

1. 首先从当前的 store 中取得缓存的 FEED_QUERY 数据。

2. 现在你获取到了用户刚刚点赞了的新闻链接。此时需要手动修改一下缓存中这个新闻链接的点赞数，让它和从服务器返回的数据一致。

3. 最后，把修改过的数据写回到 store 中。

接下来，把这个方法作为 updateStoreAfterVote 属性传递给 Link 组件。

修改 LinkList.js：

```JavaScript
<Link
  key={link.id}
  link={link}
  index={index}
  updateStoreAfterVote={this._updateCacheAfterVote}
/>
```

完成了，现在当修改了 vote 之后，update 函数将会被触发然后 store 也会跟着更新。store 的更新将会让组件重新渲染，那么在 UI 上，就可以看到最新的信息了。

下面，我们同样也让组件在新建一个新闻链接后也可以自动更新。

打开 CreateLink.js 并更新 `<Mutation />` 组件：

```js
import { FEED_QUERY } from './LinkList'

// ...

<Mutation
  mutation={POST_MUTATION}
  variables={{ description, url }}
  onCompleted={() => this.props.history.push('/')}
  update={(store, { data: { post } }) => {
    const data = store.readQuery({ query: FEED_QUERY })
    data.feed.links.unshift(post)
    store.writeQuery({
      query: FEED_QUERY,
      data
    })
  }}
>
  {postMutation => <button onClick={postMutation}>Submit</button>}
</Mutation>
```

其中，update 函数和之前的运作方式一样。首先读取 FEED_QUERY 来获取当前 state 的结果，然后在其中加入一个最新创建的 link 并写回到 store 中。

超赞！现在本地 store 也会在新建新闻链接后自动更新。我们的应用越来越有模有样了 🤓
