> * 原文地址：[Queries: Loading Links](https://www.howtographql.com/react-apollo/2-queries-loading-links/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# Query：加载链接

## 准备 React 组件

这个应用需要实现的第一个功能就是加载并显示一个 Link。我们从渲染单个 link 的组件开始，并且该 Link 元素继承自 React component。

在 components 目录下创建一个新的文件：Link.js，然后添加如下的代码：

```JavaScript
import React, { Component } from 'react'

class Link extends Component {
  render() {
    return (
      <div>
        <div>
          {this.props.link.description} ({this.props.link.url})
        </div>
      </div>
    )
  }

  _voteForLink = async () => {
    // ... you'll implement this in chapter 6
  }
}

export default Link
```

这是个非常简单的 React 组件，不再赘述。

然后在 components 目录下再创建一个新的文件：LinkList.js，并添加如下代码：

```JavaScript
import React, { Component } from 'react'
import Link from './Link'

class LinkList extends Component {
  render() {
    const linksToRender = [
      {
        id: '1',
        description: 'Prisma turns your database into a GraphQL API 😎 😎',
        url: 'https://www.prismagraphql.com',
      },
      {
        id: '2',
        description: 'The best GraphQL client',
        url: 'https://www.apollographql.com/docs/react/',
      },
    ]

    return (
      <div>{linksToRender.map(link => <Link key={link.id} link={link} />)}</div>
    )
  }
}

export default LinkList
```

这里用了本地 mock 数据来检验 Link 组件是否运作正常。后面这些数据将会从后台服务获取。

最后，修改 App.js 文件：

```JavaScript
import React, { Component } from 'react'
import LinkList from './LinkList'

class App extends Component {
  render() {
    return <LinkList />
  }
}

export default App
```

运行 app 看看是不是目前为止一切正常？如果正常的话，在浏览器应该能看到：

![graphqlpic7](../imgs/graphqlpic7.png)

## 写 GraphQL query

下面你将要从数据库加载真正的链接地址。首先你去要定义你想发送给 API 的 GraphQL query：

```JavaScript
query FeedQuery {
  feed {
    links {
      id
      createdAt
      description
      url
    }
  }
}
```

你可以在 playground 里面执行这段代码并从 GraphQL 服务得到结果。但是如何在 JS 代码中使用它呢？

## 使用 Apollo 客户端发送 query

当使用 Apollo 时，你有两种发送 query 给服务端的方式。

第一种是在 ApolloClient 直接使用 query 方法。这是种方式方式简单直接，并允许你用 promise 的方式获取结果。

例如：

```JavaScript
client.query({
  query: gql`
    query FeedQuery {
      feed {
        links {
          id
        }
      }
    }
  `
}).then(response => console.log(response.data.allLinks))
```

另一个可以搭配 React 使用的更常用的方法是采用 Apollo 的高阶组件 graphql，将 React 组件使用一个 query 包裹起来。

使用这个方法获取数据，你要做的就是写好 GraphQL query 然后 graphql 就将会帮助你获取数据然后把它作为 props 提供给组件。

我们来改写 LinkList.js：

```JavaScript
// 1
const FEED_QUERY = gql`
  # 2
  query FeedQuery {
    feed {
      links {
        id
        createdAt
        url
        description
      }
    }
  }
`

// 3
export default graphql(FEED_QUERY, { name: 'feedQuery' }) (LinkList)
```

这段代码是什么意思呢：

1. 常量 FEED_QUERY 包含了这个 query，gql 函数用来解析这个包含了 GraphQL 代码的字符串。(如果你不熟悉 '``' 语法，[参考这里](https://wesbos.com/tagged-template-literals/))

2. 现在，已经定义好了 GraphQL query，FeedQuery 是操作名，将会被 Apollo 用作该 query 的引用。（# 是 GraphQL 语言的注释符）

3. 最后，使用 graphql 来包裹 LinkList 组件， FEED_QUERY 是调用 graphql 其中的一个参数。注意到，还有另一个对象作为第二个参数，将 name 指定为 feedQuery。这个名字是 Apollo 将会作为 props 注入 LinkList 组件的名字。如果没有定义，那默认的 props 的名字就是 data。

为了让这段代码能够工作，还需要 import 相关的依赖：

```JavaScript
import { graphql } from 'react-apollo'
import gql from 'graphql-tag'
```

这就是用来获取数据的全部代码了，是不是很棒？

现在你可以删除掉那些 mock 的数据了，数据已经可以从服务端获取。

更新 LinkList.js 的 render：

```JavaScript
render() {
  // 1
  if (this.props.feedQuery && this.props.feedQuery.loading) {
    return <div>Loading</div>
  }

  // 2
  if (this.props.feedQuery && this.props.feedQuery.error) {
    return <div>Error</div>
  }

  // 3
  const linksToRender = this.props.feedQuery.feed.links

  return (
    <div>{linksToRender.map(link => <Link key={link.id} link={link} />)}</div>
  )
}
```

让我们来分析下这段代码到底做了什么？正如我们所预期的那样，Apollo 给组件注入了一个新的 prop，名字是 feedQuery，这个 prop 有三个字段，提供了有关网络请求的信息：

1. loading：当网络请求还依旧进行，还没有收到应答的时候，这个值是 true。

2. error：如果请求失败，这个字段就会包含错误信息。

3. feed：这个是从服务器接收到的返回信息。

> 实际上，对于 prop 设置还有很多，可以查看[文档](https://www.apollographql.com/docs/react/essentials/queries.html#graphql-query-data)进行更多探索。

完成了，现在再次运行 yarn start 来试试看。你将会看到真实的从服务端返回的数据了。（别忘了在 server 目录下也需要 yarn start 哦！）