> * 原文地址：[Pagination](https://www.howtographql.com/react-apollo/9-pagination/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 分页

本篇教程我们将会学习如何为数据分页，这是 React-Apollo 模块的最后一章了。我们将会实现一个简单的分页方法，这样用户就可以分段获取并查看数个新闻链接，而不是一次性获取包含所有新闻链接的列表。

## 准备 React 组件

首先，我们需要稍微调整下路由配置。LinkList 组件将会被两个路由引用。第一种情况下，它将会显示前十名投票最高的新闻链接，第二种情况下，它将会将新闻链接数据分成多页展示，并且可由用户选择页码。

修改 [App.js](https://github.com/howtographql/react-apollo/blob/master/src/components/App.js) 文件：

```JavaScript
import { Switch, Route, Redirect } from 'react-router-dom'

// ...

render() {
  return (
    <div className='center w85'>
      <Header />
      <div className='ph3 pv1 background-gray'>
        <Switch>
          <Route exact path='/' render={() => <Redirect to='/new/1' />} />
          <Route exact path='/create' component={CreateLink} />
          <Route exact path='/login' component={Login} />
          <Route exact path='/search' component={Search} />
          <Route exact path='/top' component={LinkList} />
          <Route exact path='/new/:page' component={LinkList} />
        </Switch>
      </div>
    </div>
  )
}
```

我们新增了两个路由：`/top` 和 `/new/:page`。后者的 url 中包含页码 page，路由相关的组件，即 LinkList 组件就可以读取到这个信息。

根目录 `/` 现在重定向到了页码是 1 的列表页：`/new/1`。

同时 [Header.js](https://github.com/howtographql/react-apollo/blob/master/src/components/Header.js) 也要作出相应的修改：

```JavaScript
<Link to="/top" className="ml1 no-underline black">
  top
</Link>
<div className="ml1">|</div>
```

同样我们需要修改 LinkList 组件的逻辑，来实现两个路由对应的逻辑功能。

修改 LinkList 中的 FeedQuery，为它增加三个参数：

```JavaScript
export const FEED_QUERY = gql`
  query FeedQuery($first: Int, $skip: Int, $orderBy: LinkOrderByInput) {
    feed(first: $first, skip: $skip, orderBy: $orderBy) {
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

query 请求现在可以接受用于实现分页需要的参数：skip 参数定义了偏移量，即数据应该从第多少位开始。first 用于限制数据条目的数量，即应返回多少条新闻链接。也就是，如果定义了 skip 是 10，first 是 5。那么将会返回总列表的第 10-15 个。orderBy 定义了返回的数据应当如何排序。

但是当使用 `<Query />` 组件获取数据的时候，如何将这些参数传递进去呢？你需要在声明组件的时候，通过 variables 属性提供参数：

在 LinkList 中添加如下方法：

```JavaScript
_getQueryVariables = () => {
  const isNewPage = this.props.location.pathname.includes('new')
  const page = parseInt(this.props.match.params.page, 10)

  const skip = isNewPage ? (page - 1) * LINKS_PER_PAGE : 0
  const first = isNewPage ? LINKS_PER_PAGE : 100
  const orderBy = isNewPage ? 'createdAt_DESC' : null
  return { first, skip, orderBy }
}
```

然后更新 LinkList 中的 `<Query />` 组件：

```js
<Query query={FEED_QUERY} variables={this._getQueryVariables()}>
```

现在，我们将当前页面的 first, skip, orderBy 参数作为 variables 属性传递给了组件。可以使用 `ownProps.match.params.page`（React-Router 特性）获取当前页码，然后计算出 first 和 skip 的值。

createdAt_DESC 属性则保证了最新创建的新闻链接将会优先显示。而如果组件是通过路由 `/top` 加载的，那么需要返回的是投票最高的新闻链接。

同时，需要在 constants.js 文件中定义常量 LINKS_PER_PAGE，并将其引入 LinkList.js 文件：

```js
export const LINKS_PER_PAGE = 5
```

## 不同页码间的跳转

修改 LinkList.js，添加两个用来翻页的按钮：

```JavaScript
import React, { Component, Fragment } from 'react'
import { LINKS_PER_PAGE } from '../constants'

// ...

render() {
  return (
    <Query query={FEED_QUERY} variables={this._getQueryVariables()}>
      {({ loading, error, data, subscribeToMore }) => {
        if (loading) return <div>Fetching</div>
        if (error) return <div>Error</div>

        this._subscribeToNewLinks(subscribeToMore)
        this._subscribeToNewVotes(subscribeToMore)

        const linksToRender = this._getLinksToRender(data)
        const isNewPage = this.props.location.pathname.includes('new')
        const pageIndex = this.props.match.params.page
          ? (this.props.match.params.page - 1) * LINKS_PER_PAGE
          : 0

        return (
          <Fragment>
            {linksToRender.map((link, index) => (
              <Link
                key={link.id}
                link={link}
                index={index + pageIndex}
                updateStoreAfterVote={this._updateCacheAfterVote}
              />
            ))}
            {isNewPage && (
              <div className="flex ml4 mv3 gray">
                <div className="pointer mr2" onClick={this._previousPage}>
                  Previous
                </div>
                <div className="pointer" onClick={() => this._nextPage(data)}>
                  Next
                </div>
              </div>
            )}
          </Fragment>
        )
      }}
    </Query>
  )
}
```

> 关于 React Fragment，[请戳这里](https://reactjs.org/docs/fragments.html)。

方法 _getLinksToRender 用于计算哪些新闻链接将会被展示：

```JavaScript
_getLinksToRender = data => {
  const isNewPage = this.props.location.pathname.includes('new')
  if (isNewPage) {
    return data.feed.links
  }
  const rankedLinks = data.feed.links.slice()
  rankedLinks.sort((l1, l2) => l2.votes.length - l1.votes.length)
  return rankedLinks
}
```

如果 isNewPage 为 true，那么我们不需要修改列表，直接返回所有结果即可。但如果用户加载的是 `/top` 路由，你就需要根据点赞数量将新闻链接排序。

下面我们来实现翻页相关的函数：

```JavaScript
_nextPage = data => {
  const page = parseInt(this.props.match.params.page, 10)
  if (page <= data.feed.count / LINKS_PER_PAGE) {
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

非常简单，只需要从当前页面 url 中获取页码，并计算将要跳转的页码是多少，然后使用 `this.props.history` 修改路由即可。此时页面就会重新加载，组件会再次根据路由信息获取到相应的参数。

## 其他一些代码调整

由于我们修改了 FEED_QUERY，所以此时 `store.readQuery` 也需要作出调整，否则 update 函数将无法正常工作。readQuery 现在也需要获取参数：`first, skip, orderBy`。

> 注：ApolloClient 提供的 readQuery 方法其实和 query 方法的工作原理是一样的。但是，readQuery 并没有向服务端发起请求，而只是将本地 store 更新了！如果从服务端获取数据的请求结果是带参数的，那么 readQuery 方法也需要获取这些参数，以保证它能够正确的更新缓存信息。

我们更新 _updateCacheAfterVote 方法如下：

```JavaScript
_updateCacheAfterVote = (store, createVote, linkId) => {
  const isNewPage = this.props.location.pathname.includes('new')
  const page = parseInt(this.props.match.params.page, 10)

  const skip = isNewPage ? (page - 1) * LINKS_PER_PAGE : 0
  const first = isNewPage ? LINKS_PER_PAGE : 100
  const orderBy = isNewPage ? 'createdAt_DESC' : null
  const data = store.readQuery({
    query: FEED_QUERY,
    variables: { first, skip, orderBy }
  })

  const votedLink = data.feed.links.find(link => link.id === linkId)
  votedLink.votes = createVote.link.votes
  store.writeQuery({ query: FEED_QUERY, data })
}
```

上面这段代码中，基于路由的不同（`/top` 或 `/new`），计算页码参数的方法也不同。

最后，update 方法的实现也需要调整。

打开 [CreateLink.js](https://github.com/howtographql/react-apollo/blob/master/src/components/CreateLink.js) 文件，修改 Mutation 组件：

```JavaScript
import { LINKS_PER_PAGE } from '../constants'

// ...

<Mutation
  mutation={POST_MUTATION}
  variables={{ description, url }}
  onCompleted={() => this.props.history.push('/new/1')}
  update={(store, { data: { post } }) => {
    const first = LINKS_PER_PAGE
    const skip = 0
    const orderBy = 'createdAt_DESC'
    const data = store.readQuery({
      query: FEED_QUERY,
      variables: { first, skip, orderBy }
    })
    data.feed.links.unshift(post)
    store.writeQuery({
      query: FEED_QUERY,
      data,
      variables: { first, skip, orderBy }
    })
  }}
>
  {postMutation => <button onClick={postMutation}>Submit</button>}
</Mutation>
```

现在，你已经为应用增加了一个简单的分页功能，用户可以分段查看数据，而不必一次性加载全部数据了。
