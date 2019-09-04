# Mutation介绍

mutation中文意思是改变、变异。
更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数：
```
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
```
你不能直接调用一个 mutation handler。这个选项更像是事件注册：“当触发一个类型为 increment 的 mutation 时，调用此函数。”要唤醒一个 mutation handler，你需要以相应的 type 调用 store.commit 方法：
```
store.commit('increment')
```
## Mutation的载荷（payload）
你可以向 store.commit 传入额外的参数，即 mutation 的 载荷（payload）：
```
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}
```
```
store.commit('increment', 10)
```
在大多数情况下，载荷应该是一个对象，这样可以包含多个字段并且记录的 mutation 会更易读：
```
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```
```
store.commit('increment', {
  amount: 10
})
```
## 对象风格的提交方式
提交 mutation 的另一种方式是直接使用包含 type 属性的对象：
```
store.commit({
  type: 'increment',
  amount: 10
})
```
当使用对象风格的提交方式，整个对象都作为载荷传给 mutation 函数，因此 handler 保持不变：
```
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```
## Mutation 需遵守 Vue 的响应规则

既然 Vuex 的 store 中的状态是响应式的，那么当我们变更状态时，监视状态的 Vue 组件也会自动更新。这也意味着 Vuex 中的 mutation 也需要与使用 Vue 一样遵守一些注意事项：

1.最好提前在你的 store 中初始化好所有所需属性。

2.当需要在对象上添加新属性时，你应该使用 Vue.set(obj, 'newProp', 123), 或者以新对象替换老对象。例如，利用 stage-3 的对象展开运算符我们可以这样写：
```
state.obj = { ...state.obj, newProp: 123 }
```
## Mutation 必须是同步函数
举例说明：
```
mutations: {
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```
现在想象，我们正在 debug 一个 app 并且观察 devtool 中的 mutation 日志。每一条 mutation 被记录，devtools 都需要捕捉到前一状态和后一状态的快照。然而，在上面的例子中 mutation 中的异步函数中的回调让这不可能完成：因为当 mutation 触发的时候，回调函数还没有被调用，devtools 不知道什么时候回调函数实际上被调用——实质上任何在回调函数中进行的状态的改变都是不可追踪的。

#   Vue Router介绍
Vue Router 是 Vue.js 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。包含的功能有：嵌套的路由/视图表；模块化的、基于组件的路由配置；路由参数、查询、通配符；基于 Vue.js 过渡系统的视图过渡效果；细粒度的导航控制；带有自动激活的 CSS class 的链接；HTML5 历史模式或 hash 模式，在 IE9 中自动降级；自定义的滚动条行为。

## 嵌套路由
嵌套路由和嵌套组件之间的匹配是个很常见的需求，使用 vue-router 可以很简单的完成这点。
假设我们有如下一个应用：
```
<div id="app">
  <router-view></router-view>
</div>
```
<router-view> 是一个顶级的外链。它会渲染一个和顶级路由匹配的组件：
```
router.map({
  '/foo': {
    // 路由匹配到/foo时，会渲染一个Foo组件
    component: Foo
  }
})
```
## 路由匹配
### 动态片段
动态片段使用以冒号开头的路径片段定义，例如 user/:username 中，:username 就是动态片段。它会匹配注入 /user/foo 或者 /user/bar 之类的路径。当路径匹配一个含有动态片段的路由规则时，动态片段的信息可以从 $route.params 中获得。使用示例：
```router.map({
  '/user/:username': {
    component: {
      template: '<p>用户名是{{$route.params.username}}</p>'
    }
  }
})
```
一条路径中可以包含多个动态片段，每个片段都会被解析成 $route.params 的一个键值对。
### 全匹配片段
动态片段只能匹配路径中的一个部分，而全匹配片段则基本类似于它的贪心版。例如 /foo/*bar 会匹配任何以 /foo/ 开头的路径。匹配的部分也会被解析为 $route.params 中的一个键值对。





# Vuex Todo List - v2

## 重构:

> 将目前所有数据存储都改为来自**state**的方式，所有的数据获取来源和修改来源都来自服务器端的API Call，当数据从API获取或者修改以后，再去调整**state**让UI render

**例如以下获取状态的todo list的API:**

```
http://5b4dcb2aec112500143a2311.mockapi.io/api/todos?search=active
```

**构建自己的Mcok API Server:**

## <https://www.mockapi.io/projects>

**课参考文档**：<https://www.mockapi.io/docs>

**访问HTTP API使用 npm库：** axios, 具体使用方法参考axios github文档

### 代码重构思路提示

todosAPI封装了对API的异步调用，仅仅在成功后通过回调通知，再回调中再调用dispath，例如：

```
apiUrl: 'http://5b4dcb2aec112500143a2311.mockapi.io/api';
  add(todo, successCallBack) {
    // this.todos.push(todo, successCallBack);
    axios
      .post(`${this.apiUrl}/todos`, {
        id: todo.viewId,
        content: todo.content,
        status: todo.status
      })
      .then(function(response) {
        console.log(response.data);
        successCallBack(
          new Todo(
            response.data.id,
            response.data.content,
            response.data.status
          )
        );
      })
      .catch(function(error) {
        console.log(error);
      });
  }
```

### 要求

- Git commit 小步提交
- 每次提交的评论都有意义

## 编程题答题流程

- 在命令行中使用以下命令在用户本地任意目录下clone此题目库

```
git clone repo_of_this_template
```

- 仔细阅读题目要求和题目描述（题目的README.md中），完成题目
- 在github上创建远端仓库(此处使用`your_remote_repo`代替你在github上创建的远端仓库)
- 设置远程仓库

```
git remote set-url origin your_remote_repo`
```

- 在本地使用**git提交(commit)**并**上传(push)**，之后将github仓库地址(your_remote_repo) eg:（[https://github.com/username/your_remote_repo）](https://github.com/username/your_remote_repo%EF%BC%89) 填入到提交地址一栏
- 获取分支（该题目分支填写master即可）
- 提交
- 等待结果
