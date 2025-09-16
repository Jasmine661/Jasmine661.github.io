---
title: JWT
description: 
top_img: 
cover: 
categories: 零散的小知识点
---
# 概念

即JSON Web Token，通过数字签名的方式，以JSON对象为载体，在不同服务器终端之间安全的传输信息。

# 作用

最常见的场景就是授权认证，一旦用户登录，后续每个请求都将包含JWT，系统在每次用户请求之前，都要先进行JWT安全校验，通过之后再进行处理。

# 组成

三个组成部分之间用 . 分隔

header: 用于标识JWT的类型，指定加密算法，如 HS256。
payload: 用于存储JWT的主体信息，如用户信息、权限信息等。
signature: 用于对JWT进行数字签名，防止JWT被篡改。

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.        ← Header（头部）
eyJpZCI6MTIzLCJ1c2VybmFtZSI6ImphY2sifQ.      ← Payload（载荷）
fOxy1Go-R9qJ95v8SzFY5j4q0s57HdERrkxxUK9m5a4  ← Signature（签名）
```

