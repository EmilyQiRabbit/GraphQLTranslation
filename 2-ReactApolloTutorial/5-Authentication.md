> * 原文地址：[Authentication](https://www.howtographql.com/react-apollo/5-authentication/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 认证

这一章我们将会学习如何配合 Apollo 使用认证方法来为用户提供注册登录功能。

## 准备 React 组件

和之前一样，首先需要准备好 React 组件：本节需要构建 Login 组件。

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
    const { login, email, password, name } = this.state
    return (
      <div>
        <h4 className="mv3">{login ? 'Login' : 'Sign Up'}</h4>
        <div className="flex flex-column">
          {!login && (
            <input
              value={name}
              onChange={e => this.setState({ name: e.target.value })}
              type="text"
              placeholder="Your name"
            />
          )}
          <input
            value={email}
            onChange={e => this.setState({ email: e.target.value })}
            type="text"
            placeholder="Your email address"
          />
          <input
            value={password}
            onChange={e => this.setState({ password: e.target.value })}
            type="password"
            placeholder="Choose a safe password"
          />
        </div>
        <div className="flex mt3">
          <div className="pointer mr2 button" onClick={() => this._confirm()}>
            {login ? 'login' : 'create account'}
          </div>
          <div
            className="pointer button"
            onClick={() => this.setState({ login: !login })}
          >
            {login
              ? 'need to create an account?'
              : 'already have an account?'}
          </div>
        </div>
      </div>
    )
  }

  _confirm = async () => {
    // ... you'll implement this soon
  }

  _saveUserData = token => {
    localStorage.setItem(AUTH_TOKEN, token)
  }
}

export default Login
```

快速了解下这个组件的结构，它有两个状态：

1. 第一个是用户已有账号，仅需要登录。这时候组件将会渲染两个输入框，让用户输入 email 和 password。这时候 state.login 是 true。

2. 第二个状态是用户没有账号，需要注册。这时候组件就会渲染第三个输入框让用户输入 name，这时候 state.login 是 false。

_confirm 方法将会执行 mutation，发送登录时需要提供给服务端的信息。

还需要创建一个 contants.js 文件，定义存储在 localStorage 中的登录信息的 key。

> 警告 ⚠️ ：在 localStorage 中存储敏感信息其实是非常不安全的。但是本教程的主要专注点是 GraphQL，这里就简单处理了。[这里](https://www.rdegges.com/2018/please-stop-using-local-storage/)介绍了更多的相关信息。

在 src 文件里创建 constants.js：

```JavaScript
export const AUTH_TOKEN = 'auth-token'
```

组件准备好了，在 App.js 中添加一个新的路由：

```JavaScript
import Login from './Login'

...

render() {
  return (
    <div className="center w85">
      <Header />
      <div className="ph3 pv1 background-gray">
        <Switch>
          <Route exact path="/" component={LinkList} />
          <Route exact path="/create" component={CreateLink} />
          <Route exact path="/login" component={Login} />
        </Switch>
      </div>
    </div>
  )
}
```

最后，修改 Header.js，加上和 login 相关的内容：

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

首先从 local storage 中获取 authToken，如果 authToken 不可用，那么提交按钮就不会被渲染。这就保证了只有登录过的用户可以创建新的链接。

然后在 Header 右边添加一个按钮，让用户可以用来登录和登出。

好了，组件就都准备好了～可以开始实现登录功能了。

## 认证 mutation

登录和注册的方式和刚才新建链接的方法很类似：

修改 Login.js 文件：

```JavaScript
import gql from 'graphql-tag'

const SIGNUP_MUTATION = gql`
  mutation SignupMutation($email: String!, $password: String!, $name: String!) {
    signup(email: $email, password: $password, name: $name) {
      token
    }
  }
`

const LOGIN_MUTATION = gql`
  mutation LoginMutation($email: String!, $password: String!) {
    login(email: $email, password: $password) {
      token
    }
  }
`
```

这两个 mutation 和之前的 mutation 很相似。它们需要数个参数，并返回了 token，你可以用它来和后续的请求绑定，并以此认证用户。

另外，修改 Login.js：

```js
import { Mutation } from 'react-apollo'
import gql from 'graphql-tag'

...

<div className="flex mt3">
  <Mutation
    mutation={login ? LOGIN_MUTATION : SIGNUP_MUTATION}
    variables={{ email, password, name }}
    onCompleted={data => this._confirm(data)}
  >
    {mutation => (
      <div className="pointer mr2 button" onClick={mutation}>
        {login ? 'login' : 'create account'}
      </div>
    )}
  </Mutation>
  <div
    className="pointer button"
    onClick={() => this.setState({ login: !login })}
  >
    {login ? 'need to create an account?' : 'already have an account?'}
  </div>
</div>
```

代码解析：

代码其实很简单明了。如果用户想要登录，调用 loginMutation；而如果是注册，则使用 signupMutation。mutation 依靠按钮触发。GraphQL mutation 接受 variables prop 提供的 email, password 和 name 作为参数。最后，mutation 请求结束后，我们调用 _confirm 函数：它是作为参数传递给 onCompleted prop 的。

下面，修改 Login.js 的 _confirm 方法：

```JavaScript
_confirm = async data => {
  const { token } = this.state.login ? data.login : data.signup
  this._saveUserData(token)
  this.props.history.push(`/`)
}
```

返回值中可以获取 token，我们把它保存下来，然后进行路由跳转。

## 使用认证 token 配置 Apollo

现在，用户可以登录并且获取到了 token。你需要保证，每个发送给 API 的请求都包含这个 token。

既然所有的请求都是 ApolloClient 的实例发送的，那么你就需要保证这个实例拿到了这个 token。对此，Apollo 提供了一个很好的方法：使用一个由 Apollo Link 实现的中间件。

首先，需要为 app 安装依赖，在项目根目录输入：

```js
yarn add apollo-link-context
```

打开 index.js 文件，添加代码：

```JavaScript
import { setContext } from 'apollo-link-context'

...

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem(AUTH_TOKEN)
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : ''
    }
  }
})
```

每次 ApolloClient 发送请求的时候，这个中间件都会运行一次。
Apollo Links 允许创建中间件来在发送 request 之前对它们做修改。

> 更多有关 Apollo Client links 的内容[参见这里](https://blog.graph.cool/all-you-need-to-know-about-apollo-client-2-7e27e36d62fd)

index.js 中还需要做相关的修改包括：

```JavaScript
const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache()
})
```

```JavaScript
import { AUTH_TOKEN } from './constants'
```

现在，如果 token 存在，你的所有 API 请求都将会携带该认证信息。

## 服务端认证相关

需要确保，仅有认证了的用户可以 post 新的链接。并且，每个 post 创建的 Link 都需要使用 postedBy 字段来保存用户信息。

修改 resolvers/Mutation.js 文件

```JavaScript
function post(parent, { url, description }, ctx, info) {
  const userId = getUserId(ctx)
  return ctx.db.mutation.createLink(
    { data: { url, description, postedBy: { connect: { id: userId } } } },
    info,
  )
}
```

这样，你就可以从请求的 Authorization header 中提取出 userId 并使用它来将用户和 Link 联系起来。

注意：如果 ctx 没有提供有效的 token，那么 getUserId 方法将会报错。

[self Proofreading +1]
