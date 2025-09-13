---
title: React项目配置的小tips
description: 
top_img: /image/React.jpg
cover: /image/React.jpg
---
# CSS样式

1.直接引入整个css文件

```javascript
import '.index.css'
<div className= "app"></div>
```
2.JSS模块化引入组件
```javascript
import style from './index.css'
<div className={style.app}></div>
```
# CSS的类型

把css转化为js对象，实现css的类型

## 配置

插件 typescript-plugin-css-module
注意安装为dev依赖

```javascript
// tsconfig.json 的 compilerOptions 下添加
"plugin": [
  {
    "name": "typescript-plugin-css-module"
  }
]

// .vscode/settings.json
{
  "typescript.tsdk": "node_module/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

# class类组件

基本语法结构

```javascript
import React, { Component } from 'react';

type Props = {
  name: string;
  age?: number; // 可选属性
};

type State = {
  count: number;
};

class MyComponent extends Component<Props, State> {
  constructor(props: Props) {
    super(props) // 给React Component传值
    this.state = {
      count: 0,
    }
  }

  handleClick = () => {
    this.setState({ count: this.state.count + 1 })
  }

  render() {
    const { name, age } = this.props
    const { count } = this.state

    return (
      <div>
        <h1>Hello, {name}!</h1>
        {age && <p>Age: {age}</p>}
        <p>Count: {count}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    )
  }
}

export default MyComponent
```