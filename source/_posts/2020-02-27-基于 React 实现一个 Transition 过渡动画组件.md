---
title: 基于 React 实现一个 Transition 过渡动画组件
date: 2020-02-27
toc: false
comments: false
tags:
    - React过渡动画组件
    - React组件开发
categories:
    - React
---

过渡动画使 UI 更富有表现力并且易于使用。如何使用 React 快速的实现一个 Transition 过渡动画组件？

<!--more-->

### 基本实现

实现一个基础的 CSS 过渡动画组件，通过切换 CSS 样式实现简单的动画效果，也就是通过添加或移除某个 class 样式。因此需要给 Transition 组件添加一个 toggleClass 属性，标识要切换的 class 样式，再添加一个 action 属性实现样式切换，action 为 true 时添加 toggleClass 到动画元素上，action 为 false 时移除 toggleClass。

安装 classnames 插件：
```static
npm install classnames --save-dev
```
[classnames](https://github.com/JedWatson/classnames) 是一个简单的JavaScript实用程序，用于有条件地将 className 连接在一起。

在 components 目录下新建一个 Transition 文件夹，并在该文件夹下新建一个 Transition.jsx 文件：
```js static
import React from 'react'
import classnames from 'classnames'

/**
 * css过渡动画组件
 *
 * @visibleName Transition 过渡动画
 */
class Transition extends React.Component {
  render() {
    const { children } = this.props
    const transition = (
      <div
        className={
          classnames({
            transition: true
          })
        }
        style={
          {
            position: 'relative',
            overflow: 'hidden'
          }
        }
      >
        <div
          className={
            classnames({
              'transition-wrapper': true
            })
          }
        >
          { children }
        </div>
      </div>
    )
    return transition
  }
}

export default Transition
```
这里使用了 [JSX](https://react.docschina.org/docs/introducing-jsx.html)，在 JSX 中，使用 camelCase（小驼峰命名）来定义属性的名称，使用大括号“{}”嵌入任何有效的 [JavaScript 表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions)。
如：
```js static
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;
```
等价于：
```js static
const element = <h1>Hello, Josh Perez</h1>;
```

> **注意：**
> 因为 JSX 语法上更接近 JavaScript 而不是 HTML，所以 React DOM 使用 camelCase（小驼峰命名）来定义属性的名称，而不使用 HTML 属性名称的命名约定。
> 例如，JSX 里的 class 变成了 className，而 tabindex 则变为 tabIndex。

另外，在 React 中，[props.children](https://zh-hans.reactjs.org/docs/glossary.html#propschildren) 
包含组件所有的子节点，即组件的开始标签和结束标签之间的内容（与 Vue 中 slot 插槽相似）。例如：
```html static
<Button>默认按钮</Button>
```
在 Button 组件中获取 props.children，就可以得到字符串“默认按钮”。

接下来，在 Transition 文件夹下新建一个 index.js，导出 Transition 组件：
```js static
import Transition from './Transition.jsx'

export { Transition }

export default Transition
```

然后，在 Transition.jsx 文件中为组件添加 props 检查并设置 action 的默认值：
```js static
import PropTypes from 'prop-types'

const propTypes = {
  /** 执行动画 */
  action: PropTypes.bool,
  /** 切换的css动画的class名称 */
  toggleClass: PropTypes.string
}

const defaultProps = {
  action: false
}
```
这里使用了 [prop-types](https://github.com/facebook/prop-types) 实现运行时类型检查。

>**注意：**
>prop-types 是一个运行时类型检查工具，也是 [create-react-app](https://create-react-app.dev/docs/adding-a-sass-stylesheet/) 脚手架默认配置的运行时类型检查工具，使用时直接引入即可，无需安装。

完整的 Transition 组件代码如下：
```js static
import React from 'react'
import PropTypes from 'prop-types'
import classnames from 'classnames'

const propTypes = {
  /** 执行动画 */
  action: PropTypes.bool,
  /** 切换的css动画的class名称 */
  toggleClass: PropTypes.string
}

const defaultProps = {
  action: false
}

/**
 * css过渡动画组件
 *
 * @visibleName Transition 过渡动画
 */
class Transition extends React.Component {

  static propTypes = propTypes

  static defaultProps = defaultProps

  render() {
    const {
      className,
      action,
      toggleClass,
      children
    } = this.props
    const transition = (
      <div
        className={
          classnames({
            transition: true
          })
        }
        style={
          {
            position: 'relative',
            overflow: 'hidden'
          }
        }
      >
        <div
          className={
            classnames({
              'transition-wrapper': true,
              [className]: className,
              [toggleClass]: action && toggleClass
            })
          }
        >
          { children }
        </div>
      </div>
    )
    return transition
  }
}

export default Transition
```

现在，可以使用我们的 Transition 组件了。

CSS 代码如下：
```css static
.fade {
  transition: opacity 0.15s linear;
}

.fade:not(.show) {
  opacity: 0;
}
```

JS 代码如下：
```js static
import React from 'react';
import Transition from './Transition';

class Anime extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      action: true
    }
  }
  
  render () {
    const btnText = this.state.action ? '淡出' : '淡入'
    return (
      <div>
        <Transition
          className="fade"
          toggleClass="show"
          action={ this.state.action }
        >
          淡入淡出
        </Transition>
        <button
          style={{ marginTop: '20px' }}
          onClick={() => this.setState({ action: !this.state.action })}
        >
          { btnText }
        </button>
      </div>
    )
  }
}
```
然后，在你需要该动画的地方使用 Anime 组件即可。

### 实现 Animate.css 兼容

[Animate.css](https://github.com/daneden/animate.css) 是一款强大的预设 CSS3 动画库。接下来，实现在 Transition 组件中使用 Animate.css 实现强大的 CSS3 动画。

由于 Animate.css 动画在进入动画和离开动画通常使用两个效果相反的 class 样式，因此，需要给 Transition 组件添加 enterClass 和 leaveClass 两个属性，实现动画切换。
```js static
import React from 'react'
import PropTypes from 'prop-types'
import classnames from 'classnames'

const propTypes = {
  /** 执行动画 */
  action: PropTypes.bool,
  /** 切换的css动画的class名称 */
  toggleClass: PropTypes.string,
  /** 进入动画的class名称，存在 toggleClass 时无效 */
  enterClass: PropTypes.string,
  /** 离开动画的class名称，存在 toggleClass 时无效 */
  leaveClass: PropTypes.string
}

const defaultProps = {
  action: false
}

/**
 * css过渡动画组件
 *
 * @visibleName Transition 过渡动画
 */
class Transition extends React.Component {

  static propTypes = propTypes

  static defaultProps = defaultProps

  render() {
    const {
      className,
      action,
      toggleClass,
      enterClass,
      leaveClass,
      children
    } = this.props
    return (
      <div
        className={
          classnames({
            transition: true
          })
        }
        style={
          {
            position: 'relative',
            overflow: 'hidden'
          }
        }
      >
        <div
          className={
            classnames({
              'transition-wrapper': true,
              [className]: className,
              [toggleClass]: action && toggleClass,
              [enterClass]: !toggleClass && action && enterClass,
              [leaveClass]: !toggleClass && !action && leaveClass,
            })
          }
        >
          { children }
        </div>
      </div>
    )
  }
}

export default Transition
```
>**注意：**
>由于 toggleClass 适用于那些进入动画与离开动画切换相同 class 样式的情况，而 enterClass 和 leaveClass 适用于那些进入动画与离开动画切换不同的 class 样式的情况，所以，他们与 toggleClass 不能共存。

接下来，就可以试一试加入 Animate.css 后的 Transition 组件：
```js static
import React from 'react';
import 'animate.css';

class Anime extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      action: true
    }
  }
  
  render () {
    return (
      <div>
        <Transition
          className="animated"
          enterClass="bounceInLeft"
          leaveClass="bounceOutLeft"
          action={ this.state.action }
        >
          弹入弹出
        </Transition>
        <utton
          style={{ marginTop: '20px' }}
          onClick={() => this.setState({ action: !this.state.action })}
        >
          { this.state.action ? '弹出' : '弹入' }
        </utton>
      </div>
    )
  }
}
```

### 功能扩展

通过上面的实现，Transition 组件能适用大部分场景，但是功能不够丰富。因此，接下来就需要扩展 Transition 的接口。动画通常可以设置延迟时间，播放时长，播放次数等属性。因此，需要给 Transition 添加这些属性，来丰富设置动画。

添加如下 props 属性，并设置默认值：
```js static
const propTypes = {
  ...,
  /** 动画延迟执行时间 */
  delay: PropTypes.string,
  /** 动画执行时间长度 */
  duration: PropTypes.string,
  /** 动画执行次数，只在执行 CSS3 动画时有效 */
  count: PropTypes.number,
  /** 动画缓动函数 */
  easing: PropTypes.oneOf([
    'linear',
    'ease',
    'ease-in',
    'ease-out',
    'ease-in-out'
  ]),
  /** 是否强制轮流反向播放动画，count 为 1 时无效 */
  reverse: PropTypes.bool
}

const defaultProps = {
  count: 1,
  reverse: false
}
```

根据 props 设置样式：
```js static
// 动画样式
const styleText = (() => {
  let style = {}
  // 设置延迟时长
  if (delay) {
    style.transitionDelay = delay
    style.animationDelay = delay
  }
  // 设置播放时长
  if (duration) {
    style.transitionDuration = duration
    style.animationDuration = duration
  }
  // 设置播放次数
  if (count) {
    style.animationIterationCount = count
  }
  // 设置缓动函数
  if (easing) {
    style.transitionTimingFunction = easing
    style.animationTimingFunction = easing
  }
  // 设置动画方向
  if (reverse) {
    style.animationDirection = 'alternate'
  }
  return style
})()
```

完整代码如下：
```js static
import React from 'react'
import PropTypes from 'prop-types'
import classnames from 'classnames'

const propTypes = {
  /** 执行动画 */
  action: PropTypes.bool,
  /** 切换的css动画的class名称 */
  toggleClass: PropTypes.string,
  /** 进入动画的class名称，存在 toggleClass 时无效 */
  enterClass: PropTypes.string,
  /** 离开动画的class名称，存在 toggleClass 时无效 */
  leaveClass: PropTypes.string,
  /** 动画延迟执行时间 */
  delay: PropTypes.string,
  /** 动画执行时间长度 */
  duration: PropTypes.string,
  /** 动画执行次数，只在执行 CSS3 动画时有效 */
  count: PropTypes.number,
  /** 动画缓动函数 */
  easing: PropTypes.oneOf([
    'linear',
    'ease',
    'ease-in',
    'ease-out',
    'ease-in-out'
  ]),
  /** 是否强制轮流反向播放动画，count 为 1 时无效 */
  reverse: PropTypes.bool
}

const defaultProps = {
  action: false,
  count: 1,
  reverse: false
}

/**
 * css过渡动画组件
 *
 * @visibleName Transition 过渡动画
 */
class Transition extends React.Component {

  static propTypes = propTypes

  static defaultProps = defaultProps

  render() {
    const {
      className,
      action,
      toggleClass,
      enterClass,
      leaveClass,
      delay,
      duration,
      count,
      easing,
      reverse,
      children
    } = this.props

    // 动画样式
    const styleText = (() => {
      let style = {}
      // 设置延迟时长
      if (delay) {
        style.transitionDelay = delay
        style.animationDelay = delay
      }
      // 设置播放时长
      if (duration) {
        style.transitionDuration = duration
        style.animationDuration = duration
      }
      // 设置播放次数
      if (count) {
        style.animationIterationCount = count
      }
      // 设置缓动函数
      if (easing) {
        style.transitionTimingFunction = easing
        style.animationTimingFunction = easing
      }
      // 设置动画方向
      if (reverse) {
        style.animationDirection = 'alternate'
      }
      return style
    })()

    return (
      <div
        className={
          classnames({
            transition: true
          })
        }
        style={
          {
            position: 'relative',
            overflow: 'hidden'
          }
        }
      >
        <div
          className={
            classnames({
              'transition-wrapper': true,
              [className]: className,
              [toggleClass]: action && toggleClass,
              [enterClass]: !toggleClass && action && enterClass,
              [leaveClass]: !toggleClass && !action && leaveClass,
            })
          }
          style={ styleText }
        >
          { children }
        </div>
      </div>
    )
  }
}

export default Transition
```
这里为 Transition 增加了以下设置属性：
* **delay**：规定在动画开始之前的延迟。
* **duration**：规定完成动画所花费的时间，以秒或毫秒计。
* **count**：规定动画应该播放的次数。
* **easing**：规定动画的速度曲线。
* **reverse**：规定是否应该轮流反向播放动画。

目前，Transition 的功能已经相当丰富，可以很精细的控制 CSS3 动画。

### 优化

这一步，我们需要针对 Transition 组件进一步优化，主要包括动画结束的监听、卸载组件以及兼容。

添加以下 props 属性，并设置默认值：
```js static
const propTypes = {
  ...,
  /** 动画结束的回调 */
  onEnd: PropTypes.func,
  /** 离开动画结束时卸载元素 */
  exist: PropTypes.bool
}

const defaultProps = {
  ...,
  reverse: false,
  exist: false
}
```

处理动画结束的监听事件：
```js static
/**
 * css过渡动画组件
 *
 * @visibleName Transition 过渡动画
 */
class Transition extends React.Component {

  ...

  onEnd = e => {
    const { onEnd, action, exist } = this.props
    if (onEnd) {
      onEnd(e)
    }
    // 卸载 DOM 元素
    if (!action && exist) {
      const node = e.target.parentNode
      node.parentNode.removeChild(node)
    }
  }

  /**
   * 对动画结束事件 onEnd 回调的处理函数
   *
   * @param {string} type - 事件解绑定类型: add - 绑定事件，remove - 移除事件绑定
   */
  handleEndListener (type = 'add') {
    const el = ReactDOM.findDOMNode(this).querySelector('.transition-wrapper')
    const events = ['animationend', 'transitionend']
    events.forEach(ev => {
      el[`${type}EventListener`](ev, this.onEnd, false)
    })
  }

  componentDidMount () {
    this.handleEndListener()
  }

  componentWillUnmount () {
    const { action, exist } = this.props
    if (!action && exist) {
      this.handleEndListener('remove')
    }
  }

  render () {
    ...
  }
}
```
这里使用到两个生命周期函数 componentDidMount 和 componentWillUnmount，关于 React 生命周期的介绍请移步[组件生命周期](https://react.docschina.org/docs/state-and-lifecycle.html#adding-lifecycle-methods-to-a-class)。

[react-dom](https://react.docschina.org/docs/react-dom.html) 提供了可在 React 应用中使用的 DOM 方法。

获取兼容性的 animationend 事件和 transitionend 事件。不同的浏览器要求使用不同的前缀，因为火狐和IE都已经支持了这两个事件，因此，只需针对 webkit 内核浏览器进行兼容的 webkitTransitionEnd 事件检测。检测函数代码如下：
```js static
/**
 * 浏览器兼容事件检测函数
 *
 * @param {node} el - 触发事件的 DOM 元素
 * @param {array} events - 可能的事件类型
 * @returns {*}
 */
const whichEvent = (el, events) => {
  const len = events.length
  for (var i = 0; i < len; i++) {
    if (el.style[i]) {
      return events[i];
    }
  }
}
```

修改 handleEndListener 函数：
```js static
/**
 * css过渡动画组件
 *
 * @visibleName Transition 过渡动画
 */
class Transition extends React.Component {

  ...

  /**
   * 对动画结束事件 onEnd 回调的处理函数
   *
   * @param {string} type - 事件解绑定类型: add - 绑定事件，remove - 移除事件绑定
   */
  handleEndListener (type = 'add') {
    const el = ReactDOM.findDOMNode(this).querySelector('.transition-wrapper')
    const events = ['AnimationEnd', 'TransitionEnd']
    events.forEach(ev => {
      const eventType = whichEvent(el, [ev.toLowerCase(), `webkit${ev}`])
      el[`${type}EventListener`](eventType, this.onEnd, false)
    })
  }

  ...

}
```

到这里，我们完成了整个 Transition 组件的开发，完整代码如下：
```js static
import React from 'react'
import PropTypes from 'prop-types'
import classnames from 'classnames'
import ReactDOM from 'react-dom'

const propTypes = {
  /** 执行动画 */
  action: PropTypes.bool,
  /** 切换的css动画的class名称 */
  toggleClass: PropTypes.string,
  /** 进入动画的class名称，存在 toggleClass 时无效 */
  enterClass: PropTypes.string,
  /** 离开动画的class名称，存在 toggleClass 时无效 */
  leaveClass: PropTypes.string,
  /** 动画延迟执行时间 */
  delay: PropTypes.string,
  /** 动画执行时间长度 */
  duration: PropTypes.string,
  /** 动画执行次数，只在执行 CSS3 动画时有效 */
  count: PropTypes.number,
  /** 动画缓动函数 */
  easing: PropTypes.oneOf([
    'linear',
    'ease',
    'ease-in',
    'ease-out',
    'ease-in-out'
  ]),
  /** 是否强制轮流反向播放动画，count 为 1 时无效 */
  reverse: PropTypes.bool,
  /** 动画结束的回调 */
  onEnd: PropTypes.func,
  /** 离开动画结束时卸载元素 */
  exist: PropTypes.bool
}

const defaultProps = {
  action: false,
  count: 1,
  reverse: false,
  exist: false
}

/**
 * 浏览器兼容事件检测函数
 *
 * @param {node} el - 触发事件的 DOM 元素
 * @param {array} events - 可能的事件类型
 * @returns {*}
 */
const whichEvent = (el, events) => {
  const len = events.length
  for (var i = 0; i < len; i++) {
    if (el.style[i]) {
      return events[i];
    }
  }
}

/**
 * css过渡动画组件
 *
 * @visibleName Transition 过渡动画
 */
class Transition extends React.Component {

  static propTypes = propTypes

  static defaultProps = defaultProps

  onEnd = e => {
    const { onEnd, action, exist } = this.props
    if (onEnd) {
      onEnd(e)
    }
    // 卸载 DOM 元素
    if (!action && exist) {
      const node = e.target.parentNode
      node.parentNode.removeChild(node)
    }
  }

  /**
   * 对动画结束事件 onEnd 回调的处理函数
   *
   * @param {string} type - 事件解绑定类型: add - 绑定事件，remove - 移除事件绑定
   */
  handleEndListener (type = 'add') {
    const el = ReactDOM.findDOMNode(this).querySelector('.transition-wrapper')
    const events = ['AnimationEnd', 'TransitionEnd']
    events.forEach(ev => {
      const eventType = whichEvent(el, [ev.toLowerCase(), `webkit${ev}`])
      el[`${type}EventListener`](eventType, this.onEnd, false)
    })
  }

  componentDidMount () {
    this.handleEndListener()
  }

  componentWillUnmount() {
    const { action, exist } = this.props
    if (!action && exist) {
      this.handleEndListener('remove')
    }
  }

  render () {
    const {
      className,
      action,
      toggleClass,
      enterClass,
      leaveClass,
      delay,
      duration,
      count,
      easing,
      reverse,
      children
    } = this.props

    // 动画样式
    const styleText = (() => {
      let style = {}
      // 设置延迟时长
      if (delay) {
        style.transitionDelay = delay
        style.animationDelay = delay
      }
      // 设置播放时长
      if (duration) {
        style.transitionDuration = duration
        style.animationDuration = duration
      }
      // 设置播放次数
      if (count) {
        style.animationIterationCount = count
      }
      // 设置缓动函数
      if (easing) {
        style.transitionTimingFunction = easing
        style.animationTimingFunction = easing
      }
      // 设置动画方向
      if (reverse) {
        style.animationDirection = 'alternate'
      }
      return style
    })()

    const transition = (
      <div
        className={
          classnames({
            transition: true
          })
        }
        style={
          {
            position: 'relative',
            overflow: 'hidden'
          }
        }
      >
        <div
          className={
            classnames({
              'transition-wrapper': true,
              [className]: className,
              [toggleClass]: action && toggleClass,
              [enterClass]: !toggleClass && action && enterClass,
              [leaveClass]: !toggleClass && !action && leaveClass,
            })
          }
          style={ styleText }
        >
          { children }
        </div>
      </div>
    )

    return transition
  }
}

export default Transition
```







