> * 原文地址：[Authentication](https://www.howtographql.com/react-apollo/5-authentication/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 认证

这一章我们将会学习如何配合 Apollo 使用认证方法来为用户提供注册登录功能。

## 首先还是要准备好 React 组件

在 components 目录下，创建 Login.js，然后粘贴如下代码：

```JavaScript
import React, { Component } from 'react'
import { AUTH_TOKEN } from '../constants'

class Login extends Component {
  state = {
    login: true, // switch between Login and SignUp
    email: '',
    password: '',
    name: '',
  }

  render() {
    return (
      <div>
        <h4 className="mv3">{this.state.login ? 'Login' : 'Sign Up'}</h4>
        <div className="flex flex-column">
          {!this.state.login && (
            <input
              value={this.state.name}
              onChange={e => this.setState({ name: e.target.value })}
              type="text"
              placeholder="Your name"
            />
          )}
          <input
            value={this.state.email}
            onChange={e => this.setState({ email: e.target.value })}
            type="text"
            placeholder="Your email address"
          />
          <input
            value={this.state.password}
            onChange={e => this.setState({ password: e.target.value })}
            type="password"
            placeholder="Choose a safe password"
          />
        </div>
        <div className="flex mt3">
          <div className="pointer mr2 button" onClick={() => this._confirm()}>
            {this.state.login ? 'login' : 'create account'}
          </div>
          <div
            className="pointer button"
            onClick={() => this.setState({ login: !this.state.login })}
          >
            {this.state.login
              ? 'need to create an account?'
              : 'already have an account?'}
          </div>
        </div>
      </div>
    )
  }

  _confirm = async () => {
    // ... you'll implement this in a bit
  }

  _saveUserData = token => {
    localStorage.setItem(AUTH_TOKEN, token)
  }
}

export default Login
```

快速了解下这个组件的结构，它有两个状态：

1. 第一个是用户已有账号，仅需要登陆。这时候组件将会渲染两个输入框，让用户输入 email 和 password。这时候 state.login 是 true。

2. 第二个状态是用户没有账号，需要注册。这时候组件就会渲染第三个输入框让用户输入 name，这时候 state.login 是 false。

_confirm 方法将会执行 mutation，发送登录时需要提供给服务端的信息。

还需要创建一个 contants.js 文件，定义存储在 localStorage 中的登录信息的 key。

> 警告 ⚠️ ：在 localStorage 中存储敏感信息其实是非常不安全的。但是本教程的主要专注点是 GraphQL，这里就简单处理了。[这里](https://www.rdegges.com/2018/please-stop-using-local-storage/)介绍了更多的相关信息。

在 src 文件里创建 constants.js：

```JavaScript
export const AUTH_TOKEN = 'auth-token'
```

组件准备好了，在 App.js 种添加一个新的路由：

```JavaScript
import Login from './Login'
...
render() {
  return (
    <div className="center w85">
      <Header />
      <div className="ph3 pv1 background-gray">
        <Switch>
          <Route exact path="/login" component={Login} />
          <Route exact path="/create" component={CreateLink} />
          <Route exact path="/" component={LinkList} />
        </Switch>
      </div>
    </div>
  )
}
```

最后，修改 Header.js，加上 login 相关内容：

```JavaScript
import { AUTH_TOKEN } from '../constants'
...
render() {
  const authToken = localStorage.getItem(AUTH_TOKEN)
  return (
    <div className="flex pa1 justify-between nowrap orange">
      <div className="flex flex-fixed black">
        <div className="fw7 mr1">Hacker News</div>
        <Link to="/" className="ml1 no-underline black">
          new
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
      <div className="flex flex-fixed">
        {authToken ? (
          <div
            className="ml1 pointer black"
            onClick={() => {
              localStorage.removeItem(AUTH_TOKEN)
              this.props.history.push(`/`)
            }}
          >
            logout
          </div>
        ) : (
          <Link to="/login" className="ml1 no-underline black">
            login
          </Link>
        )}
      </div>
    </div>
  )
}
```

好了，组件就都准备好了～

## 使用认证 mutation

（待续）