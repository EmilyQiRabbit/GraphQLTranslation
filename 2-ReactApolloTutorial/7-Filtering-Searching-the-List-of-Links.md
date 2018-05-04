> * 原文地址：[Filtering: Searching the List of Links](https://www.howtographql.com/react-apollo/7-filtering-searching-the-list-of-links/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 过滤器：查找特定的 Link 列表

这一章，我们将会学习如何使用 GraphQL API 来实现过滤功能

## 当然了，还是要先准备 React 元素啦

我们新建一个路由也就是新建一个页面，当然也是新的 React 元素：

在 components 文件目录下新建 Search.js 文件

```JavaScript
import React, { Component } from 'react'
import { withApollo } from 'react-apollo'
import gql from 'graphql-tag'
import Link from './Link'

class Search extends Component {

  state = {
    links: [],
    filter: ''
  }

  render() {
    return (
      <div>
        <div>
          Search
          <input
            type='text'
            onChange={(e) => this.setState({ filter: e.target.value })}
          />
          <button
            onClick={() => this._executeSearch()}
          >
            OK
          </button>
        </div>
        {this.state.links.map((link, index) => <Link key={link.id} link={link} index={index}/>)}
      </div>
    )
  }

  _executeSearch = async () => {
    // ... you'll implement this in a bit
  }

}

export default withApollo(Search)
```

熟悉的操作，标准的初始化流程。在这个 input 中用户可以输入想要查找的东西。

组件的 state 的 links 字段将会包含所有被渲染的 link，并且这次我们不能通过组件 props 来获取查询结果。我们将使用 withApollo 的方法来完成这个功能。

先吧 search 组件添加到 App.js 中：

```JavaScript
import Search from './Search'
...
render() {
  return (
    <div className='center w85'>
      <Header />
      <div className='ph3 pv1 background-gray'>
        <Switch>
          <Route exact path='/' component={LinkList}/>
          <Route exact path='/create' component={CreateLink}/>
          <Route exact path='/login' component={Login}/>
          <Route exact path='/search' component={Search}/>
        </Switch>
      </div>
    </div>
  )
}
```

在 Header.js 中也添加一下 Search 的导航：

```JavaScript
<div className="flex flex-fixed black">
  <div className="fw7 mr1">Hacker News</div>
  <Link to="/" className="ml1 no-underline black">
    new
  </Link>
  <div className="ml1">|</div>
  <Link to="/search" className="ml1 no-underline black">
    search
  </Link>
  {authToken && (
    <div className="flex">
      <div className="ml1">|</div>
      <Link to="/create" className="ml1 no-underline black">
        submit
      </Link>
    </div>
  )}
</div>
```

好了，现在组件框架搭建完毕，现在我们来实现查找功能！

## 查找 Links

在 Search.js 中加上：

```JavaScript
const FEED_SEARCH_QUERY = gql`
  query FeedSearchQuery($filter: String!) {
    feed(filter: $filter) {
      links {
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

这个 query 和 LinkLiist 的 query 很像，但是多了一个参数：filter。它将会限制你返回的 links 结果。

实际的 filter 操作在 server/src/resolvers/Query.js 文件中的 feed resolver 中实现：

```JavaScript
async function feed(parent, args, ctx, info) {
  const { filter, first, skip } = args // destructure input arguments
  const where = filter
    ? { OR: [{ url_contains: filter }, { description_contains: filter }] }
    : {}

  const queriedLinks = await ctx.db.query.links({ first, skip, where })

  return {
    linkIds: queriedLinks.map(link => link.id),
    count
  }
}
```

哎呦反正我现在看不懂，要写到 Node 教程那边才能懂吧～

大概讲一下：

定义了两个 where 条件：当这个 link 的 url 或者 description 中的一个包含了查询条件，那么这个 link 将会被返回。

完美～现在 query 都定义好了。这一次，我们希望用户按下 search 按钮后才获取数据，而不是组件一加载的时候。

这就是 withApollo 方法的目的。这个方法将 ApolloClient 实例作为 client prop 注入到了 Search 组件里。

这个 client 中有一个 query 方法，你可以用它来手动发送请求，而不是使用之前那样的 graphql 高阶组件的方法。

我们来修改下 Search.js 的 _executeSearch：

```JavaScript
_executeSearch = async () => {
  const { filter } = this.state
  const result = await this.props.client.query({
    query: FEED_SEARCH_QUERY,
    variables: { filter },
  })
  const links = result.data.feed.links
  this.setState({ links })
}
```

完成了。现在你可以手动执行 FEED_SEARCH_QUERY 然后从服务端获取结果。之后这些信息将作为组件的 state，被渲染在界面上。

用 yarn start 启动服务试试看吧！