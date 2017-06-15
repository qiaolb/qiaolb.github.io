---
layout:     post
title:      "React的Stateless Functional Components"
subtitle:   "React组件的3中定义方式"
date:       2017-02-19 00:00:00
author:     "Joe"
tags:
    - JavaScript
    - React
---

最早在写React组件时，使用的以下方法：
```javascript
var App = React.createClass({
  handleClick: function () {
    ...
  }
  
  render: function() {
    return <div onClick={handleClick}>Click me!</div>;
  }
});
```

<!-- more -->

后来使用ES6的class，写法如下：

```javascript
class App extends React.Componnet {
  handleClick() {
    ...
  }
  render() {
    return <div onClick={this.handleClick.bind(this)}>Click me!</div>
  }
}
```

今天看到另外一种写法：Stateless Functional Components，写法如下：

```javascript
function App(props) {
  function handleClick() {
    ...
  }
  return <div onClick={handleClick}>Click me!</div>
}
```

Stateless Functional Components**保持简洁和无状态**。这是**函数**，不是 Object，没有 this 作用域，是 pure function。推荐尽量使用最后一种。似乎有回到了函数年代。