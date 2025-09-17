---
title: MVC
description: 
top_img: 
cover: 
categories: 零散的小知识点
---

# 概念

MVC 是一种程序架构模式，它把程序分成三个部分：模型（Model）、视图（View）、控制器（Controller）来组织代码，从而达到分离业务操作、UI显示、逻辑控制的目的。

## View

展示层，用户交互界面，仅展示数据，不处理数据

## Model

数据层，专门负责**操作数据**，比如数据库、业务逻辑

## Controller

中间人，接收用户请求，调用模型，**返回视图**，去完成用户的请求处理，不会处理数据

# 工作流程

```sql
用户点击页面按钮（View）
   ↓
Controller（控制器）收到事件
   ↓
调用 Model（模型）处理数据
   ↓
Model 返回结果给 Controller
   ↓
Controller 渲染 View（页面显示结果）
```

# Redux和MVC的关系

其实根本没有关系，甚至是背道而驰的设计理念，为了更好的理解，可以认为，React相当于View，Redux相当于Controller，State和Reducer相当于Model