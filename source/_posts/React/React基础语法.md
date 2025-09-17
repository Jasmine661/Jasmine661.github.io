---
title: React基础语法
description: 学习React时做的小笔记呀🎄◟(˶> ᎑ <˶)◞🎄
top_img: /image/React.jpg
cover: /image/React.jpg
categories: React
---
## **useState**

用于定义，修改数据

### **原理**

流程：trigger  -> render（virtual dom） -> commit

**trigger**

在useState的set函数即将被调用时的阶段，感觉到数据要发生变化了的瞬间，即为trigger

**render**

 function重新执行的过程，函数返回值是虚拟的dom，然后对旧的虚拟dom与新的虚拟dom进行对比，发现不一样的地方

**virtual dom**

真实dom的js对象表达，即虚拟dom，有着高性能

**commit**

把虚拟dom上不同的地方反映到真实的dom上，对真实dom进行修改，页面更新

### 数据状态

useState定义的数据是**快照态的数据**，有**batchUpdate**的更新方式，即每次调用set函数时，如果是一个表达式，则会将原始设置的数据传过去，不会应为前面的set函数中调用的是函数而改变

```javascript
const [count,setCount] = useState(0)

function App() {
	const handleCount = () => {
        setCount((count) => count + 1)
        setCount(count + 1)
	}
    return <div @click={ handleCount }> { count } </div> 
} // 1
```

如果连续调用多个函数，则下次调用函数所传的参数即为上一个set函数调用函数的返回值。

useState定义的数据是组件独立的

react中的原生dom事件自带冒泡属性，可以在内部的事件中添加e.stopPropagation()阻住冒泡

## **Immutable 编程规范** 

useState定义的数据是一个immutable的数据类型，是不可以直接更改的

### **use-Immer插件**

安装后可用useImmer来定义数据（适用于复杂数据类型），其定义的数据一定是immutable的数据类型，如果直接修改，控制台会直接报错，并且还能规避许多无用的渲染

```javascript
// react官方鼓励用以下方式修改数据
const [data,setData] = useImmer({
    count: 0
})

function handleCount() {
    setCount((draft) => {
        // draft相当于将data的数据复制一份出来，形式类似与直接修改数据，但是immer底层做了分装，会帮助构建一个新对象去替换原始的数据
        // 这种写法可以有效规避immutable规范的一些错误，且写法更简单
        draft.count = draft.count + 1
    }) 
}
```

## **声明式编程及其开发规范**

1.不要定义冗余的数据

2.将相关数据聚合在一起，相关数据放在一个对象中

3.数据结构不深，如定义对象时嵌套对象

```javascript
function App() {
    const [user,setUser] = useImmer({
        firstName: '',
        lastName: ''
    })
    
    const handleFirstName = (e) => {
        setUser((draft) => {
            draft.firstName = e.target.value
        })
    }    
    
    const handleLastName = (e) => {
        setUser((draft) => {
            draft.lastName = e.target.value
        })
    }  
    
    return <div>
        <div>FirstName:<input onChange={handleFirstName}/></div>
    	<div>LastName:<input onChange={handleLastName}/></div>
    	<div>FullName: {user.firstName + ' ' + user.lastName}</div>
    </div>
}
```

## **组件通信**

子组件之间通信 -> 以父组件为桥梁

```javascript
// 父组件
import { PartOne, PartTwo} from './Parts'
import { useState } from 'react'
function App() {
  const [showPartOne,setShowPartOne] = useState(true)
  return <div>
      <PartOne showPartOne={showPartOne} setShowPartOne={setShowPartOne} />
      <PartTwo showPartOne={showPartOne} setShowPartOne={setShowPartOne} />
  </div>
}

// parts
export function PartOne({showPartOne,setShowPartOne}) {
  return <div>
    { showPartOne ? <div>PartOne</div> : null }
    <button onClick={() => setShowPartOne(true)}>Toggle</button>
  </div>
}
export function PartTwo({showPartOne,setShowPartOne}) {
  return <div>
    { !showPartOne ? <div>PartTwo</div> : null }
    <button onClick={() => setShowPartOne(false)}>Toggle</button>
  </div>
}
```

## **组件状态重置背后的运行机理**

1.UI节点位置上的内容发生改变时，原始组件会被销毁

2.组件在页面被销毁时，状态会清空

```javascript
// 父组件
import { useState } from 'react'
function App() {
  // 点击show按钮时，第二个Counter组件会被销毁，状态清空，因为ui位置内容变成了null
  const [show,setShow] = useState(true)
  // 点击showColor按钮时，Counter的颜色会发生变化，但是不会被销毁，还是Counter组件
  const [showColor,setShowColor] = useState(true)
  return <div>
      <Counter />
      {show ? <Counter /> : null}
    {showColor ? <Counter showColor={showColor} /> : <Counter />}
      <button onClick={() => {setShow(!show)}}> show </button> 
      <button onClick={() => {setShowColor(!showColor)}}> showColor </button>
  </div>
}
        
// Counter    
import { useState } from 'react'
function Counter({showColor}) {
  const [count,setCount] = useState(0)
  return <div>
    <div 
      style={{color: showColor ? 'red' : 'black' }}
      onClick={() => {setCount(count + 1)}}
    >
        {count}
    </div> 
  </div>
}
```

## **Key值的作用详解**

1. 当存在key值时，不管ui节点所在位置内容是否改变，只要key发生了变化，原始组件就会被销毁

  ```javascript
  // App
  // 当点击Click按钮时，两个Email组件的input的内容都会被清空
  // 第一个Email由于渲染结构变化（从直接渲染变为包裹在 div 中），第二个Email由于 key={to} 变化，故React会销毁并重建它们
  import Email from './Email'
  import {useState} from 'react'
  function App() {
    const [to,setTo] = useState('dell')
    return <div>
        {to === 'dell' ? <Email to={to}/> : <div><Email to={to}</div>/>}
        <Email to={to} key={to} />
      <button onClick={() => {setTo('lee')}}>Click</button>
    </div>
  }

  // Email
  function Email({to}) {
    return <div>
      To: <div>{to}</div>
      content：<input/>
    </div>
  }
  ```

2. 列表中key值的作用是提升列表渲染效率

## Reducer

可以将对数据进行加工处理的代码拆分到别的地方进行处理，相当与小型的redux，但是是react内置的

### 使用流程

- 定义数据

- 定义reducer函数

  `reducer` 是一个纯函数，它接收 **当前状态（state）** 和 **动作（action）**，并返回 **新的状态**。

- 定义action，发送改变数据的指令

- 用dispatch方法派发action

- Reducer 中 根据指令修改数据

- 完成数据的修改，return 新数据

```javascript
import { useState, useReducer } from 'react'

const listReducer = (state,action) => {
  if(action.type === 'add') {
      const newList = [...state, {
          id: action.value,
          value: action.value
      }]
      return newList
  }
  if(action.type === 'delete') {
      const newList = state.filter(item => item.id !== action.value)
      return newList
  }
  return state
}
	
function App() {
  const [inputValue,setInputValue] = useState('')
  const [list,dispatch] = useReducer(listReducer,[])
  
  const handleInputChange = (e) => {
      setInputValue(e.target.value)
  }
  
  const handleButtonClick = () => {
      const action = {
          type: 'add',
          value: inputValue
      }
      dispatch(action)
  }
  
  const handleItemClick = (id) => {
      const action = {
          type: 'delete',
          value: id
      }
      dispatch(action)
  }
  
  return <div>
    <div>
      <input value={inputValue} onChange={ handleInputChange }/>
      <button onClick={ handleButtonClick }>提交</button>
    </div>
    <ul>
      { list.map((item,index) => 
        <li 
          key={ item.id } 
          onClick={ () => { handleItemClick(item.id) } }
        >
          {item.value}
        </li>         
      )} 
    </ul>
  </div>
}
```

### 聚合数据处理逻辑

```javascript
// App
function App() {
  const [data, dispatch] = useImmerReducer(dataReducer, {
    inputValue: '',
    list: []
  });

  function handleInputChange(e){
    const action = { type: 'changeInput', value: e.target.value };
    dispatch(action);
  }

  function handleButtonClick(){
    const action = { type: 'addItem' }
    dispatch(action);
  }

  function handleItemClick(index) {
    const action = { type: 'deleteItem', value: index }
    dispatch(action);
  }
  
  return (
    <div>
      <div>
        <input value={data.inputValue} onChange={handleInputChange}/> 
        <button onClick={handleButtonClick}>提交</button>
      </div>
      <ul>
        {
          data.list.map((item, index) => (
            <li key={item.id} onClick={() => handleItemClick(index)}>
              {item.value}
            </li>
          ))
        }
      </ul>
    </div>
  )
}

export default App;

// Reducer
function dataReducer(draft, action) {
  switch(action.type) {
    case 'changeInput':
      draft.inputValue = action.value;
      return draft;
    case 'addItem':
      draft.list.push({
        id: draft.inputValue,
        value: draft.inputValue
      });
      draft.inputValue = '';
      return draft;
    case 'deleteItem':
      draft.list.splice(action.value, 1);
      return draft;
    default:
      return draft;
  }
}

export default dataReducer;
```

### 优缺点解析

- 业务逻辑代码量显著降低
- 代码的可读性比较高
- 前端自动化测试方便 Jest
- 组件的复用性会有一定的降低


## Context

是为解决组件间跨层级传递数据的问题而产生的一种数据传输方式，项目中一般用props，state就能解决问题，故Context使用较少

### 基本使用（完成深层组件传值）

```javascript
// App
import Body from './Body'
import { createContext } from 'react'

export const nameContext = createContext('') 

function App() {
  return <div>
    <nameContext.Provider value="Dell">
      <Body />
    </nameContext.Provider>
  </div>
}

// /Body/index.jsx
import Left from './Left'
import Right from './Right'
function Body() {
  return <div>
    <Left />
    <Right />
  </div>
}

// Right/index/jsx
import { useContext } from 'react'
import nameContext from '../nameContext.js'

function Right() {
  const name = useContext(nameContext)
  return <div>
    Context: {name}
  </div>
}
```

### Provider

使组件之间传递的数据更加灵活，而不是写死的最初定义的数据，甚至可以嵌套

在嵌套层级中，是向上找离自身最近的provider提供的value来使用

```javascript
// 此时Body/Right中的name的值就为helloworld
<nameContext.Provider value="Dell">
  <Header />
  <nameContext.Provider value="helloworld">
    <Body />
  </nameContext.Provider>
</nameContext.Provider>
```

通过Children进行父子组件间JSX内容的传递

### 应用场景

组件中一些内容又自己决定，一些内容由父组件决定时，可以通过children传递
组件中会有一个参数children，用来存放在父组件中为双标签呈现时，标签中嵌套的内容

```javascript
function App() {
	return <div>
    <Header >
      <div> Extor Info </div>
    </ Header>
  </div>
}

function Header({children}) {
  return <div>
    <h1>Header</h1>
    { children }
  </div>
}
```

### Reducer和Context的来联合使用

todoList示例

```javascript
import {useImmerReducer} from 'use-immer'
import {createContext,useContext} from 'react'

const DataContext = createContext({})
const DispatchContext = createContext(() => {})

const dataReducer = (draft, action) => {
  if(action.type === 'changeInput') {
    draft.inputValue = action.value
  }
  if(action.type === 'addItem') {
    draft.list.push({
      key: draft.inputValue,
      value: draft.inputValue
    })
    draft.inputValue = ''
  }
}

function App() {
  const [data,dispatch] = useImmerReducer(dataReducer,{
    inputValue: '',
    list: []
  })

  return <div>
    <DataContext.Provider value={data}>
      <DispatchContext.Provider value={dispatch}>
        <ItemList /> 
      </ DispatchContext.Provider>
    </DataContext.Provider>
  </div>
}

function ItemList() {
  const data = useContext(DataContext)
  const dispatch = useContext(DispatchContext)

  const list = data.list

  const handleChangeInput = (e) => {
    const action = {type: 'changeInput',value: e.target.value}
    dispatch(action)
  }

  const handleAddItem = () => {
    const action = { type: 'addItem'}
    dispatch(action)
  }

  return <div>
    <input value={data.inputValue} onChange={handleChangeInput}/>
    <button onClick={handleAddItem}>添加</button>
    <ul>
      {
        list.map(item => <li key={ item.key }>{ item.value }</li>)
      }
    </ul>
  </div>
}
```















