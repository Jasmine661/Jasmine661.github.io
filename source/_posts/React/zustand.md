---
title: Zustand
description: 学习React时做的小笔记呀🎄◟(˶> ᎑ <˶)◞🎄
top_img: /image/React.jpg
cover: /image/React.jpg
---
# 安装

```powershell
npm install zustand
```

# 基本实现

创建store，create的参数是一个函数，函数的返回值是对象，对象包含状态数据和方法，函数的参数set是专门用来修改数据的方法，set的参数可以是函数也可以是对象，在需要用的时候可以直接解构赋值

```javascript
import { create } from 'zustand'

const useStore = create((set) => ({
  count: 1,
  inc: () => set((state) => ({ count: state.count + 1 })),
}))

function Counter() {
  const { count, inc } = useStore()
  return (
    <div>
      <span>{count}</span>
      <button onClick={inc}>one up</button>
    </div>
  )
}
```
# 处理异步

不同于redux，zustand对于异步不用做特殊的处理，可以直接在函数中编写异步逻辑，最后只需要调用set方法传入新状态即可

```javascript
const useStore = create({
  return {
  	channelList: [],
    fetchChannelList: async () => {
      const res = await fetch(URL)
      const jsonData = await res.json()
      set({channelList: jsonData.data.channels})
    }
  }
})
```

# 切片模式

可以把不同的功能模块拆分组合，使用依旧是解构赋值

```jsx
import { create } from 'zustand'

// 创建counter相关切片
const createCounterStore = (set) => {
  return {
    count: 0,
    setCount: () => {
      set(state => ({ count: state.count + 1 }))
    }
  }
}

// 创建channel相关切片
const createChannelStore = (set) => {
  return {
    channelList: [],
    fetchGetList: async () => {
      const res = await fetch(URL)
      const jsonData = await res.json()
      set({ channelList: jsonData.data.channels })
    }
  }
}

// 组合切片
const useStore = create((...a) => ({
  ...createCounterStore(...a),
  ...createChannelStore(...a)
}))
```































