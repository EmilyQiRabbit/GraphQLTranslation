> * 原文地址：[Routing](https://www.howtographql.com/react-apollo/4-routing/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 路由

这一章我们将会学到如何将 Apollo 和 react-router 联合，来实现导航功能。

## 安装依赖

打开命令行，导航至项目根目录下，输入：

```
yarn add react-router react-router-dom
```

## 创建 Header

在为应用配置路由之前，首先需要一个 Header 元素来导航应用内的不同页面。

在 components 目录下，创建 Header.js，然后粘贴如下代码：

```JavaScript
import React, { Component } from 'react'
import { Link } from 'react-router-dom'
import { withRouter } from 'react-router'

class Header extends Component {
  render() {
    return (
      <div className="flex pa1 justify-between nowrap orange">
        <div className="flex flex-fixed black">
          <div className="fw7 mr1">Hacker News</div>
          <Link to="/" className="ml1 no-underline black">
            new
          </Link>
          <div className="ml1">|</div>
          <Link to="/create" className="ml1 no-underline black">
            submit
          </Link>
        </div>
      </div>
    )
  }
}

export default withRouter(Header)
```

这段代码渲染了两个 Link 组件，用户可以用来在 LinkList 和 CreateLink 两个组件之间导航。

> 注意，这里的 Link 组件和我们刚才写的那个 Link 组件没什么关系。这里的 Link 组件来自 'react-router-dom'，它作为链接导航不同页面。

## 配置路由 

在根元素 App 中配置路由。

打开 App 文件，然后修改 render 函数：

```JavaScript
render() {
  return (
    <div className="center w85">
      <Header />
      <div className="ph3 pv1 background-gray">
        <Switch>
          <Route exact path="/" component={LinkList} />
          <Route exact path="/create" component={CreateLink} />
        </Switch>
      </div>
    </div>
  )
}
```

别忘了还需要引入依赖：

```JavaScript
import Header from './Header'
import { Switch, Route } from 'react-router-dom'
```

下面需要将 App 组件用 BrowserRouter 包起来，这样它的子组件才能访问路由相关的方法。在 index.js 中引入依赖然后修改 render：

```JavaScript
import { BrowserRouter } from 'react-router-dom'
...
ReactDOM.render(
  <BrowserRouter>
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  </BrowserRouter>,
  document.getElementById('root'),
)
```

现在，再次运行 app，你现在可以看到两个路由了，路由 http://localhost:3000/ 将会渲染组件 LinkList，路由 http://localhost:3000/create 则会渲染 CreateList。

## 应用导航

本节的任务：在 mutation 请求生效之后，完成 CreateLink 到 LinkList 的路由跳转：

更新 CreateLink.js 的 <Mutation /> 组件

```JavaScript
<Mutation
  mutation={POST_MUTATION}
  variables={{ description, url }}
  onCompleted={() => this.props.history.push('/')}
>
  {postMutation => <button onClick={postMutation}>Submit</button>}
</Mutation>
```

[self Proofreading +1]
