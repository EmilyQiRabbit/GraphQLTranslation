> * 原文地址：[Mutations: Creating Links](https://www.howtographql.com/react-apollo/3-mutations-creating-links/)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/GraphQLTranslation)
> * 译者：[Yuqi🌸](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 🙋‍♀️🎉

# 使用 Mutations 创建 Link

这一章你将会学习如何使用 Apollo 发送 mutation（修改/新建/删除）请求，它与 query 请求的区别不大，同样也是遵循三个步骤，只是后两步稍有区别：

1. 使用 gql 解析函数写一个表示 mutation 的 JS 常量

2. 使用 <Mutation /> 组件，并将 mutation 和其他必需参数作为 props 传递

3. 使用 mutation 函数，它被注入了组件的 render prop function 中

## 准备 React 组件

首先需要准备一个能够让用户用来添加链接的 React 组件。

在 component 目录下创建一个新的文件 CreateLink.js。然后把下面的代码复制进去：

```js
import React, { Component } from 'react'

class CreateLink extends Component {
  state = {
    description: '',
    url: '',
  }

  render() {
    const { description, url } = this.state
    return (
      <div>
        <div className="flex flex-column mt3">
          <input
            className="mb2"
            value={description}
            onChange={e => this.setState({ description: e.target.value })}
            type="text"
            placeholder="A description for the link"
          />
          <input
            className="mb2"
            value={url}
            onChange={e => this.setState({ url: e.target.value })}
            type="text"
            placeholder="The URL for the link"
          />
        </div>
        <button onClick={`... you'll implement this 🔜`}>Submit</button>
      </div>
    )
  }
}

export default CreateLink
```

组件内包含了两个 input 输入框，用户可以用它们来输入想要创建的 link 的地址信息 url 和描述信息 description。信息将会被保存在 state 中，将会在用户发起 mutation 请求的时候被使用。

## 写 mutation

到底应该怎么发起 mutation 请求呢。我们来按照前文提到的三个步骤进行。

首先需要定义 mutation 代码，然后将组件用 graphql 容器包裹。这个步骤和 query 请求方法很类似。

在 CreateLing.js 中添加如下的代码：

```JavaScript
const POST_MUTATION = gql`
  mutation PostMutation($description: String!, $url: String!) {
    post(description: $description, url: $url) {
      id
      createdAt
      url
      description
    }
  }
`

...

<Mutation mutation={POST_MUTATION} variables={{ description, url }}>
  {() => (
    <button onClick={`... you'll implement this soon`}>
      Submit
    </button>
  )}
</Mutation>
```

代码解析：

1. 常量 POST_MUTATION 内包含了 mutation 信息

2. 将 button 元素作为 render prop function 的结果，包裹在 <Mutation /> 组件中，并将 POST_MUTATION 作为 prop 传递进去

3. 最后，将 description 和 url 作为 variables prop 传递进 <Mutation /> 组件

当然别忘了添加依赖：

```js
import { Mutation } from 'react-apollo'
import gql from 'graphql-tag'
```

最后，修改 <Mutation /> 组件：

```js
<Mutation mutation={POST_MUTATION} variables={{ description, url }}>
  {postMutation => <button onClick={postMutation}>Submit</button>}
</Mutation>
```

跟之前说的一样，你需要做的就仅仅是在按钮点击的时候，调用 Apollo 已经注入到 <Mutation /> 组件中的 postMutation 方法。

查看下 mutation 是否工作了，将 App.js 的 render 改成下面这样：

```js
render() {
  return (
    <CreateLink />
  )
}
```

当然记得在 App.js 中引用下 CreateLink：

```js
import CreateLink from './CreateLink'
```

现在，运行 yarn start，就可以试着发送 mutation 了。

[self Proofreading +1]