> * 原文地址：[Filtering: Searching the List of Links](https://www.howtographql.com/react-apollo/7-filtering-searching-the-list-of-links/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 条件过滤：查找符合条件的新闻链接列表

这一章，我们将会学习使用 GraphQL API 的条件过滤，并实现查找功能。

## 准备 React 元素

搜索功能需要一个新的路由地址，因此我们也需要实现一个新的 React 组件。

在 src/components 文件目录下新建 Search.js 文件，然后添加如下代码：

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
            onChange={e => this.setState({ filter: e.target.value })}
          />
          <button onClick={() => this._executeSearch()}>OK</button>
        </div>
        {this.state.links.map((link, index) => (
          <Link key={link.id} link={link} index={index} />
        ))}
      </div>
    )
  }

  _executeSearch = async () => {
    // ... 我们稍后实现这部分代码
  }
}

export default withApollo(Search)
```

这个流程我们很熟悉了。在这个 input 输入框中用户可以输入想要查找的字符串。

组件 state 中的 links 字段将会包含所有需要被渲染的新闻链接信息，所以现在我们不能通过组件的属性来获取查询结果。我们稍后将会讨论一下导出组件时使用的 withApollo 函数！

先吧 search 组件添加到 App.js 中，为其添加一个新的路由：

```JavaScript
import Search from './Search'

// ...

render() {
  return (
    <div className='center w85'>
      <Header />
      <div className='ph3 pv1 background-gray'>
        <Switch>
          <Route exact path='/' component={LinkList} />
          <Route exact path='/create' component={CreateLink} />
          <Route exact path='/login' component={Login} />
          <Route exact path='/search' component={Search} />
        </Switch>
      </div>
    </div>
  )
}
```

在 Header.js 中也添加一下 Search 页面的导航，这样用户使用起来会比较方便。

修改 [Header.js](https://github.com/howtographql/react-apollo/blob/master/src/components/Header.js):

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

好了，现在我们回到 Search 组件，来实现查找功能！

## 查询新闻链接

在 [Search.js](https://github.com/howtographql/react-apollo/blob/master/src/components/Search.js) 中加上 query 请求的定义：

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

这个 query 和 LinkList 组件中的 query 类似，但是多了一个名为 filter 的参数。它将会作为条件，限制服务端返回的列表中的新闻链接。

实际的条件过滤操作在 [server/src/resolvers/Query.js](https://github.com/howtographql/react-apollo/blob/master/server/src/resolvers/Query.js) 文件中的 feed resolver 函数中实现：

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

> 哎呦反正现在是看不懂，要学习 Node 教程的章节才可以哦。

这段代码中，定义了两个 where 条件：当这个新闻链接的 url 或者 description 中的一个包含了条件 filter 字符串，那么这个新闻链接才会被返回。两个条件 `{ url_contains: filter }` 和 `{ description_contains: filter }` 使用 Prisma 的 OR 操作符链接。

现在 query 已经定义好了，非常完美。这一次，我们希望在用户按下搜索按钮后加载数据，而不是在组件初始化加载的时候。

因此我们需要使用 withApollo 方法。这个方法将 ApolloClient 实例作为 client 属性注入到了 Search 组件里。

这个 client 属性中有一个 query 方法，你可以用它来手动发送请求，而不是像之前那样使用 graphql 高阶组件。

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

非常轻松就完成了。现在你可以手动发起 FEED_SEARCH_QUERY 请求并从服务端获取结果。之后这些信息将会存入组件的 state，然后被渲染在界面上。

用 yarn start 启动服务试试看吧！

附上 Search.js 的完整代码：

```js
import React, { Component } from 'react'
import { withApollo } from 'react-apollo'
import gql from 'graphql-tag'
import Link from './Link'

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

class Search extends Component {
  state = {
    links: [],
    filter: '',
  }

  render() {
    return (
      <div>
        <div>
          Search
          <input
            type="text"
            onChange={e => this.setState({ filter: e.target.value })}
          />
          <button onClick={() => this._executeSearch()}>OK</button>
        </div>
        {this.state.links.map((link, index) => (
          <Link key={link.id} link={link} index={index} />
        ))}
      </div>
    )
  }

  _executeSearch = async () => {
    const { filter } = this.state
    const result = await this.props.client.query({
      query: FEED_SEARCH_QUERY,
      variables: { filter },
    })
    const links = result.data.feed.links
    this.setState({ links })
  }
}

export default withApollo(Search)
```
