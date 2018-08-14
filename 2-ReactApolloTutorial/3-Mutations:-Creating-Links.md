> * 原文地址：[Mutations: Creating Links](https://www.howtographql.com/react-apollo/3-mutations-creating-links/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[旺财](https://github.com/EmilyQiRabbit)
> * **Proofreading is welcomed** 🙋 🎉

# 使用 Mutations 创建 Link

这一章你将会学习如何使用 Apollo 发送 mutation，相比于 query 它并不难，同样有三步，相比而言只有第三步稍微复杂：

1. 写一个表示 mutation 的 JS 常量，然后使用 gql 函数解析它

2. 使用 graphql 函数将组件和 mutation 包起来

3. 在组建中利用 props 来发起 mutation 请求

## 准备 React 组件

首先需要准备一个能够让用户用来添加链接的 React 组件。

在 component 目录下创建一个新的文件 CreateLink.js。然后把下面的代码放进去：

``` JavaScript
import React, { Component } from 'react'

class CreateLink extends Component {
  state = {
    description: '',
    url: '',
  }

  render() {
    return (
      <div>
        <div className="flex flex-column mt3">
          <input
            className="mb2"
            value={this.state.description}
            onChange={e => this.setState({ description: e.target.value })}
            type="text"
            placeholder="A description for the link"
          />
          <input
            className="mb2"
            value={this.state.url}
            onChange={e => this.setState({ url: e.target.value })}
            type="text"
            placeholder="The URL for the link"
          />
        </div>
        <button onClick={() => this._createLink()}>Submit</button>
      </div>
    )
  }

  _createLink = async () => {
    // ... you'll implement this in a bit
  }
}

export default CreateLink
```

组建内包含了两个 input 输入框，用户可以用它们来输入想要创建的 link 的地址信息 url 和描述信息 description。信息将会被保存在 state 中，当提交信息时，将会在 _createLink 函数中被使用。

## 写 mutation

到底应该怎么发请求呢。我们来按照前文提到的三个步骤进行。

首先需要定义 mutation 代码，然后将组件和 mutation 用 graphql 函数包裹。这个步骤和 query 请求方法很类似。

在 CreateLing.js 种添加如下的代码：

```JavaScript
// 1
const POST_MUTATION = gql`
  # 2
  mutation PostMutation($description: String!, $url: String!) {
    post(description: $description, url: $url) {
      id
      createdAt
      url
      description
    }
  }
`

// 3
export default graphql(POST_MUTATION, { name: 'postMutation' })(CreateLink)
```

代码解析：

1. 常量 POST_MUTATION 内包含了 mutation

2. mutation 包含了两个参数，url 和 description，当发起 mutation 的时候，你需要提供这两个参数

3. 最后，使用 graphql 包裹 POST_MUTATION 和 CreateLink 组件，这一次 postMutation 将会被作为 prop 传入组件，并且它是一个可以发起请求的方法

当然别忘了添加依赖：

```JavaScript
import { graphql } from 'react-apollo'
import gql from 'graphql-tag'
```

完善下 _createLink 方法，就可以看到 mutation 被发起：

```JavaScript
_createLink = async () => {
  const { description, url } = this.state
  await this.props.postMutation({
    variables: {
      description,
      url
    }
  })
}
```

跟之前说的一样，你仅仅需要调用 Apollo 注入到组件中的函数，并提供用户输入的值作为参数。

查看下 mutation 是否工作了，将 App.js 的 render 改成下面这样：

```JavaScript
render() {
  return (
    <CreateLink />
  )
}
```

当然记得在 App.js 种引用下 CreateLink：

```JavaScript
import CreateLink from './CreateLink'
```

现在，运行 yarn start，就可以试着发送 mutation 了。

[self Proofreading +1]