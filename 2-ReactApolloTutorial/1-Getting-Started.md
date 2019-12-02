> * 原文地址：[Getting Started](https://www.howtographql.com/react-apollo/1-getting-started/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# React - Apollo 入门教程

由于这部分是前端教程，所以我们就不在后端服务的实现上花费时间讲解了。我们直接使用 [Node 教程](https://www.howtographql.com/graphql-js/0-introduction)中的代码所提供服务。

这样当你创建了 React 应用，就可以直接拉取后端所需的代码。

> 提示：本教程的最终项目代码可以在 [Github](https://github.com/howtographql/react-apollo) 上找到。如果你在接下来章节的学习过程中遇到了困惑，那么随时可以参考它。同时提醒大家，每个代码块前面都标注有文件名，它直接链接到 GitHub 上对应的文件里，这样你就能清楚的知道当前代码应该处于什么位置，以及最终完成的时候它应该是什么样子。

## 前端部分

### 创建应用

首先我们创建一个 React 项目。我们使用 create-react-app 命令来完成这个任务。

当然你需要首先使用 yarn 来安装 create-react-app：

```shell
yarn global add create-react-app
```

> 注：本教程使用 [Yarn](https://yarnpkg.com/lang/en/) 来做依赖管理。你可以在 [这里](https://yarnpkg.com/en/docs/install#mac-stable) 找到安装 Yarn 的说明。如果你更喜欢使用 npm，只需运行和教程中等价的 npm 命令即可。

接下来，就可以使用它来构建你的 React 应用了：

```shell
create-react-app hackernews-react-apollo
```

这个命令会为你创建一个名为 `hackernews-react-apollo` 的目录，在这个目录下，所有基本的设置都已配置好了。

一切完成后，就可以像如下这样可以进入目录并启动项目：

```sh
cd hackernews-react-apollo
yarn start
```

这将会打开浏览器并导航至 http://localhost:3000，我们的应用就运行在这个端口。如果一切顺利，你将会看到：

![graphqlpic6](../imgs/graphqlpic6.png)

为了让项目的结构更加合理，我们在 src 文件中再创建两个子目录。一个名为 components，用来保存所有 React 组件。另一个是 styles，用来保存所有的 CSS 文件。

App.js 本身就是一个组件，所以我们把它放入 components 文件中。App.css 和 index.css 都是样式文件，所以把它们放入 styles 文件中。然后我们还需要修改对这些文件的引用：

```js
import React from 'react'
import ReactDOM from 'react-dom'
import './styles/index.css'
import App from './components/App'
```

```js
import React, { Component } from 'react';
import logo from '../logo.svg';
import '../styles/App.css';
```

你的文件目录现在应该是这样子的：

```
.
├── README.md
├── node_modules
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
├── src
│   ├── App.test.js
│   ├── components
│   │   └── App.js
│   ├── index.js
│   ├── logo.svg
│   ├── registerServiceWorker.js
│   └── styles
│       ├── App.css
│       └── index.css
└── yarn.lock
```

### 样式

本篇文章的重点在于 GraphQL 的概念以及如何在 React 应用中使用 GraphQL，所以样式问题我们就简略带过，不会花费很多时间。为了减少项目中的 CSS 代码，我们使用 [Tachyons](http://tachyons.io) 库，它提供了很多现成的 CSS 类。

打开 public/index.html 然后添加一个 link 标签引入 Tachyons：

```html
<link rel="manifest" href="%PUBLIC_URL%/manifest.json">
<link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
<link rel="stylesheet" href="https://unpkg.com/tachyons@4.2.1/css/tachyons.min.css"/>
```

但在项目中你依旧需要一些自定义样式，所以我们也为你准备了一些可能需要的其他样式：

打开 index.css 文件，添加如下代码：

```css
body {
  margin: 0;
  padding: 0;
  font-family: Verdana, Geneva, sans-serif;
}

input {
  max-width: 500px;
}

.gray {
  color: #828282;
}

.orange {
  background-color: #ff6600;
}

.background-gray {
  background-color: rgb(246,246,239);
}

.f11 {
  font-size: 11px;
}

.w85 {
  width: 85%;
}

.button {
  font-family: monospace;
  font-size: 10pt;
  color: black;
  background-color: buttonface;
  text-align: center;
  padding: 2px 6px 3px;
  border-width: 2px;
  border-style: outset;
  border-color: buttonface;
  cursor: pointer;
  max-width: 250px;
}
```

### 安装 Apollo 客户端

下一步我们要安装 Apollo 客户端（以及和 React 的绑定），这需要我们安装多个包，在命令行执行：

```shell
yarn add apollo-boost react-apollo graphql
```

我们来简单了解下这几个已经安装的包：

* [apollo-boost](https://github.com/apollographql/apollo-client/tree/master/packages/apollo-boost) 通过将 Apollo 客户端需要的几个安装包结合起来，从而为开发者提供了一些便利：
  * apollo-client：All the magic happens here ✨
  * apollo-cache-inmemory：推荐用来缓存的包
  * apollo-link-http：用于获取服务端数据
  * apollo-link-error：处理错误
  * apollo-link-state：状态管理
  * graphql-tag：用于 queriy 和 mutation 的 gql 方法

* [react-apollo](https://github.com/apollographql/react-apollo) 包含了 Apollo 与 React 的绑定功能

* [graphql](https://github.com/graphql/graphql-js) 包含了 Facebook 对 GraphQL 的实现，Apollo 客户端也需要用到它的部分功能。

好了，现在我们可以开始写代码了！🚀

### 配置 ApolloClient

Apollo 将底层的网络逻辑抽象隐藏了，并为 GraphQL 服务提供了易于对接的接口。相比于使用 REST API，你不需要构建 HTTP 请求了 - 你只需要写好 query 和 mutation，然后用 ApolloClient 实例发送即可。

首先你需要完成 ApolloClient 实例的配置。它需要知道 GraphQL API 的端口，以便于建立网络连接。

打开 src/index.js，写入如下代码：

```JavaScript
import React from 'react'
import ReactDOM from 'react-dom'
import './styles/index.css'
import App from './components/App'
import * as serviceWorker from './serviceWorker';

// 1
import { ApolloProvider } from 'react-apollo'
import { ApolloClient } from 'apollo-client'
import { createHttpLink } from 'apollo-link-http'
import { InMemoryCache } from 'apollo-cache-inmemory'


// 2
const httpLink = createHttpLink({
  uri: 'http://localhost:4000'
})

// 3
const client = new ApolloClient({
  link: httpLink,
  cache: new InMemoryCache()
})

// 4
ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  document.getElementById('root')
)
serviceWorker.unregister();
```

> 注：使用 create-react-app 生成的项目代码使用了分号，并且使用双引号表示字符串。而上文的代码则没有使用分号，并且使用单引号表示字符串。你可以按照你的习惯，将代码中的分号删除，并将双引号替换为单引号。🔥

我们来解释下这段代码：

1. 从已经安装的包中引入需要的依赖

2. 创建 HttpLink，它会将 ApolloClient 实例和 GraphQL API 连接起来。你的 GraphQL 服务应当运行在 http://localhost:4000。

3. 通过传入 httpLink 和 InMemoryCache 实例，实例化 ApolloClient。

4. 渲染 React 应用的 root 组件。该组件需要被高阶组件 ApolloProvider 包裹，并将 ApolloClient 的实例作为 client 属性传入。

好了，一切配置完毕，现在开始向应用中传入一些数据吧！😎

## 后端

### 下载服务端代码

就像前面说的，后端部分，我们就直接用 [Node 教程](https://www.howtographql.com/graphql-js/0-introduction) 提供的项目代码。

在 hackernews-react-apollo 目录下运行：

```sh
curl https://codeload.github.com/howtographql/react-apollo/tar.gz/starter | tar -xz --strip=1 react-apollo-starter/server
```

>如果你使用的是 windows 系统，你需要安装 [Git CLI](https://git-scm.com) 来避免 curl 命令可能有的坑。

现在在文件中可以看到一个新的 server 文件，这里面包含了所有你需要的后端代码。

代码的解释这里就不详述了，就将安装过程写一下，后面的 Node 教程会对这里进行详细解说。

### 部署 Prisma 数据库服务

这是启动服务前的最后一步了：部署数据库。

具体的方式就是，安装依赖然后激活部署。在控制台，首先进入 server 文件目录下，然后执行：

```
cd server
yarn install
yarn prisma deploy
```

> 当执行 yarn prisma deploy 的时候，随便选择一个开发 cluster 就可以，public-us1 或者 public-eu1 均可。

执行成功后，在打印的日志里面将 HTTP 端口信息粘贴到 server/src/index.js 文件里，替换掉 `__PRISMA_ENDPOINT__`：

```js
const server = new GraphQLServer({
  typeDefs: './src/schema.graphql',
  resolvers,
  context: req => ({
    ...req,
    db: new Prisma({
      typeDefs: 'src/generated/prisma.graphql',
      endpoint: '__PRISMA_ENDPOINT__',
      secret: 'mysecret123',
    }),
  }),
})
```

> 如果日志丢失，可以执行：

```
yarn prisma info
```

### “探索”服务

现在你可以开始试着探索你的服务了。在 server 目录下，执行如下命令来启动服务：

```
yarn start
```

这行命令将会运行 package.json 文件中 script start 对应的命令。它首先会启动服务，然后将会打开一个 [GraphQL 练习场](https://github.com/graphcool/graphql-playground)，你可以用它来探索 API。

如果启动报错，请更新你的 node 版本。

如果你打开了 playground，你就可以试着输入一些代码来测试了，参照[原文](https://www.howtographql.com/react-apollo/1-getting-started/)的栗子很简单～如果有兴趣的可以去看看，这里不赘述。
