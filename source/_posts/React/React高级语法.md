---
title: React高级语法
description: 学习React时做的小笔记呀🎄◟(˶> ᎑ <˶)◞🎄
top_img: /image/React2.jpg
cover: /image/React2.jpg
categories: React
---
# React高级语法

## useRef

用来保存 React 组件中不需要驱动页面变更的数据，即引用一个不需要渲染的值

### 操作ref引用一个值

```javascript
import { useRef, useState } from 'react';
function App() {
  const [ time, setTime ] = useState((new Date()).getTime());
  const timer = useRef(null);
  
  function handleStartClick() {
    timer.current = setInterval(() => {
      setTime((new Date()).getTime());
    }, 1000)
  }

  function handleStopClick() {
    clearInterval(timer.current);
  }

  return (
    <div>
      <button onClick={handleStartClick}>Start</button>
      <button onClick={handleStopClick}>Stop</button>
      <div>{time}</div>
    </div>
  )
}
export default App;
```

### 通过 ref 操作 DOM

#### 操作元素节点

```javascript
import { useRef } from 'reacts'
function App() {
  const inputValue = useRef(null)
  function handleInputFocus() {
    inputValue.current.focus()
  }
  return <div>
    <input ref={ inputValue }/>
    <button onClick={ handleInputFocus }>Focus</button>
  </div>
}
```

#### 操作组件

尽量少使用，会加重代码的维护负担
步骤：父组件定义ref -> 把ref传递给子组件 -> 子组件通过forwardRef获取父组件传递过来的ref -> 把ref绑定到子组件的一个dom节点上 -> 绑定成功（示例中的inputElement.current就是input这个dom节点）

```javascript
import { useRef, forwardRef } from 'react'

function App() {
  const inputElement = useRef(null)
  function handleInputFocus() {
    inputElement.current.focus()
  }
  return <div>
    <InputComponent ref={ inputElement }/>
    <button onClick={ handleInputFocus }>Focus</button>
  </div>
}

const InputComponent = forwardRef((props,ref) => {
  return (<input ref={ref} />)
})
```

## useEffect

### 生命周期

创建执行 -> 销毁
组件中Effect的挂载和销毁会执行很多次，并且都是在render之后执行的

### 使用步骤

1.使用useEffect定义，此时，每次render过后，其第一个参数的回调函数会被执行
2.第二个参数可以限定Effect hook的执行时机，参数值时数组，当为空数组时，只有在第一次渲染的时候回调函数会执行，当内容为额定依赖项时，依赖项发生变化的时候，回调函数执行

```javascript
import { useEffect, useState } from 'react'

function App() {
  const [name,setName] = useSate('Dell')

  useEffect({fetch.get('http://localhost:3000/a.json')
    .then(console.log('123')).catch(console.log('abc'))
  },[name])

  return <div>
    <div>{name}</div>
  </div>
}
```
### 底层执行逻辑

1. 清除Effect引入的临时内容，避免内存泄漏
```javascript
import { useEffect, useState } from 'react'

function App() {
  const [show,setShow] = useState(false)

  function handleShowBtn() {
    setShow(!show)
  }

  return <div>
    { show ? <Timer /> : null}
    <button onClick={ handleShowBtn }>Toggle</button>
  </div>
}

function Timer() {
  const [time,setTime] = useState((new Date()).getTime())

  useEffect(() => {
    const timer = setInterval(() => {
      setTime((new Date()).getTime())
    },1000)
    return () => {
      console.log('clear process......')
      clearInterval(timer)
    }
  })

  return <div>{time}</div>
}
```
2.Effect并不是跟随render顺序执行的，而是在render执行后，页面更新完成之后再执行的
下面代码运用Effect可以实现再点击toggle之后播放video，再点击时停止播放
```javascript
function App() {
  const [isPlaying,setIsPlaying] = useState(false)

  function handleIsPlaying() {
    setIsPlaying(!isPlaying)
  }

  return <div>
    <VideoPlayer src='...' isPlaying={isPlaying} />
    <button onClick={handleIsPlaying}>Toggle</button>
  </div>
}

function VideoPlayer ({src,isPlaying}) {
  const ref = useRef(null)

  useEffect(() => {
    if(isPlaying) {
      ref.current.play()
    }else {
      ref.current.pause()
    }
  })

  return <div>
    <video ref={ref} src={src} loop />
  </div>
}
```
### strict mode(严格模式)

用于辅助开发，在严格模式下进行本地开发的时候，会故意重复挂载和卸载组件，即render函数会执行两次，是因为react希望我们的render函数时一个纯函数，即有固定的输入和固定的输出，看执行函数两次的时候，返回的内容和只执行一次有什么区别，如果有区别，页面就会出bug

```javascript
// 用React.StrictMod标签包裹App组件，则会开启严格模式
<React.StrictMode>
  <App />
</React.StrictMode>
```
**在严格模式下，Effect的执行逻辑**
如果不进行对Effect中的定时器、全局事件、dom操作等不稳定的依赖项和副作用进行清理，可以帮助你提前发现潜在问题，确保代码更健壮
定时器
```javascript
function App() {
  const [ time, setTime ] = useState((new Date()).getTime())
  useEffect(() => {
    const timer = setInterval(() => {
      setTime((new Date()).getTime())
    }, 1000);
    return () => {
      clearInterval(timer);
    }
  }, [])

  return <div style={{height: '5000px'}}>{time}</div>
}
```
DOM操作
```javascript
function App() {
  const [ time, setTime ] = useState((new Date()).getTime())
  // DOM 操作在严格模式下，需要进行清理
  useEffect(() => {
    const originBackground = window.document.body.style.background;
    window.document.body.style.background = 'red';
    return () => {
      window.document.body.style.background = originBackground;
    }
  }, []);

  return <div style={{height: '5000px'}}>{time}</div>
}
```
全局事件
```javascript
function App() {
  const [ time, setTime ] = useState((new Date()).getTime())
  // 全局事件绑定在严格开发模式下，需要进行清理
  useEffect(() => {
    function onScroll() {
      console.log('scroll');
    }
    window.addEventListener('scroll', onScroll);
    
    return () => {
      window.removeEventListener('scroll', onScroll)
    }
  }, []);

  return <div style={{height: '5000px'}}>{time}</div>
}
```
## useMemo
在每次重新渲染的时候能够缓存计算的结果，相当于vue中的计算属性
### Memo的使用
示例：
```javascript
function App() {
  const list = ['Do homework', 'Clean Rooms', 'Coding', 'Watering Flower'];
  const [name, setName] = useState('');
  const [search, setSearch] = useState('');

  const filteredList = list.filter(item => item.indexOf(search) > -1)

  return (
    <div>
      <div>
        name: <input value={name} onChange={(e)=>{setName(e.target.value)}}/>
      </div>
      <div>
        search: <input value={search}onChange={(e)=>{setSearch(e.target.value)}}/>
      </div>
      <div>
        <select>
          {filteredList.map(item => {
            return <option key={item}>{item}</option>
          })}
        </select>
      </div>
    </div>
  );
}
```
以上代码中改变name的值时，filteredList的计算逻辑会被无用的执行（name时State定义的，改变时会执行render），存在性能不高的问题
用以下代码代替filteredList的计算逻辑可以很好的避免性能问题，useMemo相当于是做了一个缓存，即filteredList所依赖的数据发生变化的情况下，filteredList才会重新计算
```javascript
const filteredList = useMemo(() => {
    return list.filter(item => item.indexOf(search) > -1);
  // eslint-disable-next-line
  }, [search]);
```
与Effect写法类似，但Memo是在render执行的同时执行，而Effect是在render结束之后执行的

## useSyncExternalStore

主要用于跟外部系统进行对接，使用不多，但简洁明了

### 语法

**useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)**
subscribe 订阅函数监听外部存储的变化
getSnapshot 决定从外部系统的哪里获取数据，当数据发生变化的时候,React会重新执行render

```javascript
const [isOnline,setIsOnline] = useState(true)
useEffect(() => {
  window.addEventListener('online', () => {setIsOnline(window.navigator.onLine)})
  window.addEventListener('offline', () => {setIsOnline(window.navigator.onLine)})
  return () => {
    window.addEventListener('online', () => {setIsOnline(window.navigator.onLine)})
    window.addEventListener('offline', () => {setIsOnline(window.navigator.onLine)})
  }
},[])

// 以上代码可以替换为：
// 模拟外部
const subscribe(callback) {
  window.addEventListener('online', callback)
  window.addEventListener('offline', callback)
  return () => {
    window.removeEventListener('online', callback)
    window.removeEventListener('offline', callback)
  }
}
// 组件内
const isOnline = useSyncExternalStore(subscribe, () => window.navigator.onLine)
```

## useEffectEvent

实验性API，能解决一些过度依赖的问题，用于简化useEffect的依赖管理，即useEffectEvent允许你在 useEffect 中使用一个函数，该函数引用稳定（不触发重新执行），但内部总能访问最新的状态和 props，从而减少不必要的依赖项。

以下代码中，request没有被useEffectEvent包裹时，useEffect中是要求要多加一个param的依赖项的，但是加了之后
```javascript
function App() {
  const [url, setUrl] = useState('http://localhost:3000')
  const [param, setParam] = useState('?name=dell')
  const request = useEffectEvent((url) => {
    console.log(`发送请求，地址是${url}${param}`)
  })
  useEffect(() => { request(url);}, [url])
  return (
    <>
      <div onClick={()=>{setUrl('http://localhost:3001')}}>Change Url</div>
      <div onClick={()=>{setParam('?name=lee')}}>Change Param</div>
    </>
  );
}
```
### 自定义Hook

提升代码复用性，把分散的逻辑聚集在一起，代码可读性更好。
Hook共享的时执行的逻辑，多次执行同一个Hook，每次执行对应的状态不是被共享的。

```javascript
function useUserInfo() {
  const [ userInfo, setUserInfo ] = useState({})
  useEffect(() => {
    setUserInfo({name: 'dell', job: 'teacher'})
  }, [])
  function changeUserInfo() {
    setUserInfo({name: 'lee', job: "engineer"})
  }
  return [ userInfo, changeUserInfo ]
}

function App() {
  const [ userInfo, changeUserInfo ] = useUserInfo()
  const [ userInfoOne, changeUserInfoOne ] = useUserInfo()
  return (
    <>
      <div onClick={changeUserInfo}>
        {userInfo.name}
      </div>
      <div onClick={changeUserInfoOne}>
        {userInfoOne.name}
      </div>
    </>
  );
}
```
## useCallback

避免render过程中反复重新生成函数

```javascript
function App() {
  const [content, setContent] = useState('')

  const handleContentChange = useCallback((e) => {
    setContent(e.target.value)
  }, [])

  return (
    <input value={content ? content: ''} onChange={handleContentChange}/>
  )
}
```
## useDebugValue

类似于console.log,用于对自定义Hook的调试

```javascript
useDebugValue('...打印的内容')
```
## useImperativeHandle

父组件调用子组件DOM时，能对DOM节点上的返回内容做一层中转限制,父组件能调用的东西只能从useImperativeHandle钩子return的内容中选择

```javascript
const UserInput = forwardRef((props, ref) => {
  const inputRef = useRef(null)
  useImperativeHandle(ref, () => {
    // 父组件能调用的东西只能从return的内容中选择
    return {
      blur() {
        inputRef.current.blur()
      },
      value: inputRef.current.value
    }
  }, [])

  const [ value, setValue ] = useState('dell')
  return <input ref={inputRef} value={value || ''} onChange={(e)=>{setValue(e.target.value)}} />
})


function App() {
  const ref = useRef(null);

  useEffect(() => {
    console.log(ref.current.value);
  }, [])

  return (
    <UserInput ref={ref} />
  )
}
```
## useDeferredValue 

用于延迟更新 UI 的某些部分

### 使用

1.使用Suspense组件时，不显示备选方案，在组件加载完成之后直接渲染
原页面显示：dell -> loading...(备选方案，此时lee正在加载) -> lee -> loading -> dell .....
使用后：dell -> (在加载lee的时候，继续显示dell) -> lee

```javascript
const Todos = lazy(() => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(import('./Todos'))
    }, 1000)
  })
})

const Hello = lazy(() => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(import('./Hello'))
    }, 1000)
  })
})

function App() {
  const [ isHello, setIsHello ] = useState(true)
  const deferredIsHello = useDeferredValue(isHello)

  const Component = deferredIsHello ? Hello : Todos

  return (
    <>
      <button onClick={()=>{setIsHello(false)}}>
        Toggle
      </button>
      <Suspense fallback={<div>Loading...</div>}>
        <Component />
      </Suspense>
    </>
  )
}
```
2.在要渲染的组件性能很低的时候，直接的用户交互可能会导致界面卡顿
用useDeferredValue创造一个延迟版本的inputValue——deferredInputValue，React会在浏览器空闲时或在更紧急的更新完成后才应用这个延迟值到Todos组件
```javascript
const Todos = memo(({text}) => {
  const items = [];
  for(let i = 0; i < 100; i++) {
    items.push(<div key={i}>{text}</div>)
  }
  const startTime = (new Date()).getTime()
  // 模拟低性能
  while((new Date()).getTime() - startTime < 60) {}
  return <div>{items}</div>
});

function App() {
  const [ inputValue, setInputValue ] = useState('');
  const deferredInputValue = useDeferredValue(inputValue);

  return (
    <>
      <input
        value={inputValue || ''}
        onChange={(e) => {setInputValue(e.target.value)}}
      />
      <Todos text={deferredInputValue} />
    </>
  );
}
```

## useTransition

创建一个延迟执行的流程，等浏览器空闲的时候再去执行延迟执行流程里的内容(startTransition函数的函数体)

```javascript
const Todos = memo(({text}) => {
  const items = []
  for(let i = 0; i < 100; i++) {
    items.push(<div key={i}>{text}</div>)
  }
  const startTime = (new Date()).getTime();
  while((new Date()).getTime() - startTime < 60) {}
  return <div>{items}</div>
})

function App() {
  const [ inputValue, setInputValue ] = useState('')
  const [ deferredInputValue, setDeferredInputValue ] = useState('')
  const [ isPending, startTransition ] = useTransition()

  function handleOnChange(e) {
    setInputValue(e.target.value)
    startTransition(() => {
      setDeferredInputValue(e.target.value)
    })
  }
  return (
    <>
      <input
        value={inputValue || ''}
        onChange={handleOnChange}
      />
      { isPending ? <div>Loading...</div> : <Todos text={deferredInputValue} />}
    </>
  );
}
```
# React内置组件
## Profiler
用于对组件性能的优化，检测渲染性能
```javascript
// 使用
<Profiler id="App" onRender={onRender}>
  <App />
</Profiler>
// onRender回调函数
function onRender(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  // 对渲染时间进行汇总或记录...
  // phase: "mount","update","nested-update"
  // actualDuration 组件实际渲染时间
  // baseDuration 参考值，一般多久渲染完毕
  // startTime 真正开始渲染的时间
}
```
## Suspense
当组件还没加载的时候，展示一个备选方案，在组件能被渲染的时候再渲染，还没弄懂
```javascript
<Suspense fallback={<Loading />}>
  <App />
</Suspense>
```

# API

## memo

组件所依赖的props没有发生变化时，通过memo定义的组件不渲染，使用缓存
解决当State定义的值发生改变时，所有的子组件都会重新渲染导致的性能问题。

### 语法

**memo(Component, arePropsEqual?)**
Component是要进行记忆化的组件
arePropsEqual是一个函数，返回值为false不使用缓存，true使用缓存，相当与自定义。函数的两个参数是oldProps和newProps

```javascript
const Child = memo(({ name, address }) =>  {
  console.log('Child render');
  return <div>{name}{address}</div>
}, (originProps, props) => {
  if(originProps.address !== props.address) {
    return false;
  }
  return true;
})

function App() {
  const [ name, setName ] = useState('');
  const [ address, setAddress ] = useState('');
  return (
    <>
      <div>
        name: <input value={name || ''} onChange={(e) => {setName(e.target.value)}} />
      </div>
      <div>
        address: <input value={address || ''} onChange={(e) => {setAddress(e.target.value)}} />
      </div>
      <Child name={name} address={address}></Child>
    </>
  );
}
```