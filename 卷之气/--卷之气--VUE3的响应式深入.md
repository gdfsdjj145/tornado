### VUE3的响应式深入

- 副作用effect（核心中的核心）
  - effect，一个正在运行的函数
  - effect储存栈堆

```js
// 维持一个执行副作用的栈
const runningEffects = []

const createEffect = fn => {
  // 将传来的 fn 包裹在一个副作用函数中
  const effect = () => {
    runningEffects.push(effect)
    fn()
    runningEffects.pop()
  }

  // 立即自动执行副作用
  effect()
}
```

当我们的副作用被调用时，在调用 `fn` 之前，它会把自己推到 `runningEffects` 数组中。这个数组可以用来检查当前正在运行的副作用。

- 跟踪变化

  - 跟踪变化功臣-- [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

    **Proxy 是一个对象，它包装了另一个对象，并允许你拦截对该对象的任何交互**

    ```js
    // 常规使用proxy
    const nei = {
        juan: '内卷'
    }
    
    // 控制函数
    const handler = {
        get(target, property) {
            console.log('老子卷四你们')
            return target([property])
        }
    }
    // 没有对象就new一个对象
    const proxy = new Proxy(nei, handler)
    console.log(proxy.juan)
    // 老子卷四你们
    // 内卷
    ```

  - 绑定this指向功臣--[Reflect](https://www.bookstack.cn/read/es6-3rd/spilt.1.docs-reflect.md)

    ES6新增特性`Reflect` 可以帮我将任何的事情，就在`Reflect`上进行

    ```js
    // 常规使用proxy
    const nei = {
        juan: '内卷'
    }
    
    // 控制函数
    const handler = {
        get(target, property, receiver) {
            console.log('老子卷四你们')
          	return Reflect.get(...arguments)
        }
    }
    // 没有对象就new一个对象
    const proxy = new Proxy(nei, handler)
    console.log(proxy.juan)
    // 老子卷四你们
    // 内卷
    ```

  - 神秘的侦探--`Track`

    ```JS
    const handler = {
      get(target, property, receiver) {
        track(target, property)
        return Reflect.get(...arguments)
      }
    }
    ```

    `track`函数可以检测到当前是哪个`effect`在运行，并将对象和处理方法记录下来，意思就是你读取啥数据，我帮你找出来

  - 执行者--`trigger`

    当检测的值被修改时，我们就要运行该值的修改方法，vue3运用了一个`trigger`函数来做这件事

    ```js
    const handler = {
      get(target, property, receiver) {
        track(target, property)
        return Reflect.get(...arguments)
      },
      set(target, property, value, receiver) {
        trigger(target, property)
        return Reflect.set(...arguments)
      }
    }
    ```

    值改变，就会执行`set`方法，`trigger`就会寻找那些`effect`是当前这个值的并执行

- 渲染

​		当追踪的对象数值发生改变的时候，会执行对应的`effect`的渲染函数，重新生成`render`



##### 最后附带以前vue2的响应式原理深入[vue2](https://www.yuque.com/docs/share/0011c8b3-0af9-448c-841c-4a5ffc2fc4a8?# 《Vue的响应式原理》)

--卷之气--Vue的响应式分析