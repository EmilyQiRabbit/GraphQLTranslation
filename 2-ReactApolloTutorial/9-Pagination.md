> * 原文地址：[Pagination](https://www.howtographql.com/react-apollo/9-pagination/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 分页

这一章的讲分页，这也是 React-Apollo 这一模块的最后一小节了～加油哈～

## 首先还是要准备 React 组件

这里，我们需要微微调整下路由。LinkList 组件将会被两个路由引用。第一种路由引用的情况下，它将会显示前十名投票最高的 link，第二种引用它将会将 link 分成多页，可由用户选择页码。

修改 App.js 文件：

```JavaScript
import { Switch, Route, Redirect } from 'react-router-dom'
...
render() {
  return (
    <div className='center w85'>
      <Header />
      <div className='ph3 pv1 background-gray'>
        <Switch>
          <Route exact path='/' render={() => <Redirect to='/new/1' />} />
          <Route exact path='/login' component={Login} />
          <Route exact path='/create' component={CreateLink} />
          <Route exact path='/search' component={Search} />
          <Route exact path='/top' component={LinkList} />
          <Route exact path='/new/:page' component={LinkList} />
        </Switch>
      </div>
    </div>
  )
}
```

top 和 new/:page 是本次新添的路由。后者的 url 中包含页码，相关的组件也就是 LinkList 就可以读取到这个信息，并根据页码作出反应。

根目录现在重定向到了页码是 1 的列表页。

同时，也对应的修改 header：

```JavaScript
<Link to="/top" className="ml1 no-underline black">
  top
</Link>
<div className="ml1">|</div>
```

下面，需要修改 LinkList 中的 FeedQuery，为它增加三个参数：

```JavaScript
export const FEED_QUERY = gql`
  query FeedQuery($first: Int, $skip: Int, $orderBy: LinkOrderByInput) {
    feed(first: $first, skip: $skip, orderBy: $orderBy) {
      count
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
      count
    }
  }
`
```

skip 参数定义了返回的信息应该从第多少个开始。first 则表示应返回多少个 link。也就是，如果定义了 skip 是 10，first 是 5。那么将会返回总列表的第 10-15 个。orderBy 定义了返回结果应当如何排序。

但是当使用 graphql 高阶组件的时候，如何将这些参数传入进去呢？你需要在使用 graphql 包裹组件的时候就提供参数：

```JavaScript
import { LINKS_PER_PAGE } from '../constants'
...
export default graphql(FEED_QUERY, {
  name: 'feedQuery',
  options: ownProps => {
    const page = parseInt(ownProps.match.params.page, 10)
    const isNewPage = ownProps.location.pathname.includes('new')
    const skip = isNewPage ? (page - 1) * LINKS_PER_PAGE : 0
    const first = isNewPage ? LINKS_PER_PAGE : 100
    const orderBy = isNewPage ? 'createdAt_DESC' : null
    return {
      variables: { first, skip, orderBy },
    }
  },
})(LinkList)
```

现在，graphql 的参数中增加了一个 options 函数，它的入参 ownProps 就是组件 props。你可以使用 ownProps.match.params.page（React-Router 特性）来获取当前页码，然后计算 first 和 skip 的值。

createdAt_DESC 属性保证了最新创建的 link 将会优先显示。而如果组件是被 /top 路由加载的，那么需要返回的是投票最高的 link。

## 实现方法

修改 LinkList.js，并添加两个用来翻页的按钮：

```JavaScript
render() {

  if (this.props.feedQuery && this.props.feedQuery.loading) {
    return <div>Loading</div>
  }

  if (this.props.feedQuery && this.props.feedQuery.error) {
    return <div>Error</div>
  }

  const isNewPage = this.props.location.pathname.includes('new')
  const linksToRender = this._getLinksToRender(isNewPage)
  const page = parseInt(this.props.match.params.page, 10)

  return (
    <div>
      <div>
        {linksToRender.map((link, index) => (
          <Link
            key={link.id}
            updateStoreAfterVote={this._updateCacheAfterVote}
            index={index}
            link={link}
          />
        ))}
      </div>
      {isNewPage &&
      <div className='flex ml4 mv3 gray'>
        <div className='pointer mr2' onClick={() => this._previousPage()}>Previous</div>
        <div className='pointer' onClick={() => this._nextPage()}>Next</div>
      </div>
      }
    </div>
  )

}
```

方法 _getLinksToRender：

```JavaScript
_getLinksToRender = (isNewPage) => {
  if (isNewPage) {
    return this.props.feedQuery.feed.links
  }
  const rankedLinks = this.props.feedQuery.feed.links.slice()
  rankedLinks.sort((l1, l2) => l2.votes.length - l1.votes.length)
  return rankedLinks
}
```

isNewPage 参数不同，返回的结果需要作出调整。如果是 /top 加载了组件，需要根据投票对 link 的顺序作出调整。

下面来实现翻页的相关函数：

```JavaScript
_nextPage = () => {
  const page = parseInt(this.props.match.params.page, 10)
  if (page <= this.props.feedQuery.feed.count / LINKS_PER_PAGE) {
    const nextPage = page + 1
    this.props.history.push(`/new/${nextPage}`)
  }
}

_previousPage = () => {
  const page = parseInt(this.props.match.params.page, 10)
  if (page > 1) {
    const previousPage = page - 1
    this.props.history.push(`/new/${previousPage}`)
  }
}
```

非常简单，只需要从现在的路由中获取页码然后直接修改路由即可。此时页面就会重新加载，然后获取到相应的内容。

## 其他代码的调整

需要更新 _updateCacheAfterVote 方法，因为修改了 FEED_QUERY，所以此时 store 的 readQuery 方法现在也希望得到 variables 参数。

> readQuery essentially works in the same way as the query method on the ApolloClient that you used to implement the search. However, instead of making a call to the server, it will simply resolve the query against the local store! If a query was fetched from the server with variables, readQuery also needs to know the variables to make sure it can deliver the right information from the cache.(啊？我在偷懒吗？那你就读一读英语吧～挺好的～You Will Like Them 😜)

```JavaScript
_updateCacheAfterVote = (store, createVote, linkId) => {
  const isNewPage = this.props.location.pathname.includes('new')
  const page = parseInt(this.props.match.params.page, 10)
  const skip = isNewPage ? (page - 1) * LINKS_PER_PAGE : 0
  const first = isNewPage ? LINKS_PER_PAGE : 100
  const orderBy = isNewPage ? 'createdAt_DESC' : null
  const data = store.readQuery({ query: FEED_QUERY, variables: { first, skip, orderBy } })

  const votedLink = data.feed.links.find(link => link.id === linkId)
  votedLink.votes = createVote.link.votes
  store.writeQuery({ query: FEED_QUERY, data })
}
```

CreateLink.js 文件也需要作出调整，因为它也用到了 store.readQuery：

```JavaScript
import { LINKS_PER_PAGE } from '../constants'
...
_createLink = async () => {
  const { description, url } = this.state
  await this.props.postMutation({
    variables: {
      description,
      url,
    },
    update: (store, { data: { post } }) => {
      const first = LINKS_PER_PAGE
      const skip = 0
      const orderBy = 'createdAt_DESC'
      const data = store.readQuery({
        query: FEED_QUERY,
        variables: { first, skip, orderBy },
      })
      data.feed.links.splice(0, 0, post)
      data.feed.links.pop()
      store.writeQuery({
        query: FEED_QUERY,
        data,
        variables: { first, skip, orderBy },
      })
    },
  })
  this.props.history.push(`/new/1`)
}
```

[self Proofreading +1]
