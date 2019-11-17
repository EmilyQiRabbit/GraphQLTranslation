> * 原文地址：[More Mutations and Updating the Store](https://www.howtographql.com/react-apollo/6-more-mutations-and-updating-the-store/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 更多 mutation 请求的实例，以及 store 的更新

下面我们来实现为链接点赞的功能。登录了的用户将可以为 link 点赞，票数比较高的链接将会被展示在一个独立的页面中。

## 准备 React 组件

第一步要做的依旧是准备好 React 组件。

打开 Link.js 文件，更新 render 方法并且记得引用需要的文件：

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

上面这段代码的 Link，能够展示出每个 link 获得的点赞个数以及创建该 link 的用户名。如果用户登陆了，就会显示点赞按钮。如果这个 link 不知道是谁创建的，用户名将会显示 Unknown。

注意到，还有一个方法 timeDifferenceForDate，传入的参数是链接创建的时间，这个方法将会吧时间戳转化为比较友好的展示方式，比如：三个小时前。

这个方法源码写在 scr 文件的 util.js 文件里，如下：

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
  }

  else if (elapsed < milliSecondsPerHour) {
    return Math.round(elapsed/milliSecondsPerMinute) + ' min ago'
  }

  else if (elapsed < milliSecondsPerDay ) {
    return Math.round(elapsed/milliSecondsPerHour ) + ' h ago'
  }

  else if (elapsed < milliSecondsPerMonth) {
    return Math.round(elapsed/milliSecondsPerDay) + ' days ago'
  }

  else if (elapsed < milliSecondsPerYear) {
    return Math.round(elapsed/milliSecondsPerMonth) + ' mo ago'
  }

  else {
    return Math.round(elapsed/milliSecondsPerYear ) + ' years ago'
  }
}

export function timeDifferenceForDate(date) {
  const now = new Date().getTime()
  const updated = new Date(date).getTime()
  return timeDifference(now, updated)
}
```

最后，每个 Link 元素也需要显示出它在列表中的序列号，所以我们将 index 从 LinkList 传递到 Link 中。

打开 LinkList.js 文件，更新 render 函数中 Link 组件的渲染代码，以加上 link 的序列号。

```JavaScript
return (
  <div>
    {linksToRender.map((link, index) => (
      <Link key={link.id} index={index} link={link} />
    ))}
  </div>
)
```

另外 LinkList 中的 FEED_QUERY 也需要修改，加入 votes 字段，获取以往点赞的用户信息和创建链接的用户。

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

好了现在元素部分就都准备好了，下面进入 vote mutation 的实现!

## 调用 mutation

在 Link.js 中添加如下和 mutation 相关的代码：

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

这个操作大家都很熟悉了。✌️ 通过添加 <Mutation /> 组件，存在于该组建内部函数的组件获得了调用 voteMutation 的能力。

运行 yarn start，试着点赞，然后刷新页面，就可以看到结果了。

下面我们来学如何在点赞后让页面自动更新。

## 更新缓存

Apollo 可以帮助你很方便的控制缓存的内容；它可以用在发起 mutation 之后更新界面。它能让你来精确定义如何更新缓存。现在我们将试着使用它，在用户发起点赞后，让 UI 界面显示出正确的赞数。

我们将会使用 Apollo 的 caching data 来完成这项功能。

更新 Link 文件的 <Mutation /> 组件，添加一个 update 属性：

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

上面添加给 update 的函数将会在服务器返回应答之后被执行。它的参数是 mutation 的 payload (也就是 data) 以及现在的缓存 store。你可以使用它们来更新缓存状态。

注意，此时你已经更新了服务器上的信息，并取回了 vote 信息。

下面真正的更新操作将会在 Link 的父级元素 LinkList 里完成——打开 LinkList.js 文件并添加如下代码：

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

2. 现在你获取到了用户刚刚点赞了的 link，就需要修改一下缓存中这个 link 的 vote，让它和从服务器返回的 votes 的结构一致。

3. 最后，把它写回到 store 中。

然后，把这个方法作为 prop 传递给 Link 组件。

修改 LinkList.js：

```JavaScript
<Link
  key={link.id}
  link={link}
  index={index}
  updateStoreAfterVote={this._updateCacheAfterVote}
/>
```

完成了，现在当修改了 vote 之后，update 将会被触发然后 store 也会跟着更新。store 的更新将会让组件重新渲染，在 UI 上，就可以看到最新的信息了。

下面，我们同样也让组件在新建 link 后自动更新。

打开 CreateLink.js 并更新 <Mutation /> 组件：

```js
import { FEED_QUERY } from './LinkList'
...
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

[self Proofreading +1]
