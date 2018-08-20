> * 原文地址：[More Mutations and Updating the Store](https://www.howtographql.com/react-apollo/6-more-mutations-and-updating-the-store/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 更多 mutation 实例以及 store 的更新

下面我们来实现为链接投票的功能。票数比较高的链接将会被展示在一个单独的页面。

## 还是要先准备好 React 组件

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

上面这段代码的 Link，能够展示出投票的个数以及创建链接的用户名。如果用户登陆了，就会显示投票按钮。如果不知道是谁创建的，用户名将会显示 Unknown。

注意到，还有一个方法 timeDifferenceForDate，传入的参数是链接创建的时间，这个方法将会吧时间戳转化为比较友好的展示方式，比如：三个小时前。

这个方法源码写在 scr 文件的 util,js 文件里，如下：

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

LinkList.js 文件也需要更新一下以提供给子元素更多的信息：

```JavaScript
return (
  <div>
    {linksToRender.map((link, index) => (
      <Link key={link.id} index={index} link={link} />
    ))}
  </div>
)
```

另外 LinkList 中的 FEED_QUERY 也需要修改，加入 votes 字段获取以往投票的用户信息和创建链接的用户。

```JavaScript
export const FEED_QUERY = gql`
  query FeedQuery {
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

好了现在元素部分就都准备好了，下面进入 vote mutation!

## 调用 mutation

在 Link.js 中添加如下 mutation 相关的代码并修改 export：

```JavaScript
import { graphql } from 'react-apollo'
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

export default graphql(VOTE_MUTATION, {
  name: 'voteMutation',
})(Link)
```

这个操作大家都很熟悉了。✌️

最后，修改 _voteForLink：

```JavaScript
_voteForLink = async () => {
  const linkId = this.props.link.id
  await this.props.voteMutation({
    variables: {
      linkId,
    },
  })
}
```

运行 yarn start，试着投票，然后刷新页面，就可以看到结果了。

下面我们来学如何自动更新。

## 更新缓存

Apollo 可以帮助你控制缓存的内容。并且非常方便，尤其是，它可以用在发起 mutation 之后。它能让你来精确定义如何更新缓存。现在我们将试着使用它，在用户发起投票后，让 UI 界面显示出正确的投票个数。

我们将会使用 Apollo 的 store API 来完成这项功能。

更新 Link 文件：

```JavaScript
_voteForLink = async () => {
  const linkId = this.props.link.id
  await this.props.voteMutation({
    variables: {
      linkId,
    },
    update: (store, { data: { vote } }) => {
      this.props.updateStoreAfterVote(store, vote, linkId)
    },
  })
}
```

上面添加的这个 update 参数将会在服务器返回应答之后被执行。它的参数是 mutation 的 payload (也就是 data) 以及现在的缓存 store。你可以使用它们来更新缓存状态。

注意，此时你已经更新了服务器上的信息，并取回了 vote 信息。

下面真正的更新操作将会在 Link 的父级元素 LinkList 里完成：

```JavaScript
_updateCacheAfterVote = (store, createVote, linkId) => {
  // 1
  const data = store.readQuery({ query: FEED_QUERY })

  // 2
  const votedLink = data.feed.links.find(link => link.id === linkId)
  votedLink.votes = createVote.link.votes

  // 3
  store.writeQuery({ query: FEED_QUERY, data })
}
```

这段代码干嘛了？

1. 首先从当前的 store 中取得了缓存的 FEED_QUERY 数据。

2. 现在你获取到了用户刚刚投票了的 link，就需要修改一下缓存中这个 link 的 vote，让它和从服务器返回的 votes 的结构一致。

3. 最后，把它写回到 store 中。

最后，吧这个方法作为 prop 传递给 Link 组件。

修改 LinkList.js：

```JavaScript
<Link key={link.id} updateStoreAfterVote={this._updateCacheAfterVote} index={index} link={link}/>
```

完成了，现在当修改了 vote 之后，update 将会被触发然后 store 也会跟着更新。store 的更新将会让组件重新渲染，在 UI 上，就可以看到最新的信息了。

[self Proofreading +1]
