---
title: flask 跨域资源共享
date: 2022-04-20
tags: [python, flask, http, cors]
---

# flask 跨域资源共享

最近完成 python 实验尝试使用了 flask 作为后端框架, 第一次用这么轻的框架从零开始搭建项目, 遇到了一点跨域上的小问题

## Flask

> Flask is a lightweight [WSGI](https://wsgi.readthedocs.io/) web application framework. It is designed to make getting started quick and easy, with the ability to scale up to complex applications. It began as a simple wrapper around [Werkzeug](https://werkzeug.palletsprojects.com/) and [Jinja](https://jinja.palletsprojects.com/) and has become one of the most popular Python web application frameworks.

flask 是在 github 上有 58k star (2022.04.20) 的开源 python 后端框架, 以超轻量级而出名, 一个用 flask 写的 hello world 甚至只需要五行代码: 

```python
# save this as app.py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello, World!"
```

使用这样的 "微框架" 可以更好地了解一些底层原理, 这里记录一次跨域资源请求问题的解决

## CORS: 跨源资源共享

### 涉及跨域资源请求的情景: 

- 由 [`XMLHttpRequest`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest) 或 [Fetch APIs](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API) 发起的跨源 HTTP 请求。
- Web 字体 (CSS 中通过 `@font-face` 使用跨源字体资源)，[因此，网站就可以发布 TrueType 字体资源，并只允许已授权网站进行跨站调用](https://www.w3.org/TR/css-fonts-3/#font-fetching-requirements)。
- [WebGL 贴图](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/Tutorial/Using_textures_in_WebGL)
- 使用 [`drawImage`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/drawImage) 将 Images/video 画面绘制到 canvas。
- [来自图像的 CSS 图形](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Shapes/Shapes_From_Images)

### 三种跨域请求:

#### [简单请求](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS#简单请求)

( 点击标题查看MDN文档 ↑↑↑ )

使用 GET/POST 方法, 手动设置的 header 字段只有以下四种, 且 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 的值有额外要求: 

- [`Accept`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept)
- [`Accept-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language)
- [`Content-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Language)
- [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) :
  - `text/plain`
  - `multipart/form-data`
  - `application/x-www-form-urlencoded`

此时服务端需要设置 header : [`Access-Control-Allow-Origin`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) , 为 * 表示允许任意外域访问该资源

#### [预检请求](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS#预检请求)

( 点击标题查看MDN文档 ↑↑↑ )

非简单请求需要预检, 要求必须首先使用 [`OPTIONS`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS) 方法发起一个预检请求到服务器, 依据服务器是否允许访问该资源决定是否发送实际请求 (通常由浏览器自动发送预检请求)

预检过程中,  预检请求中会携带了下面两个header : 

```
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

分别为 实际请求的Method, 实际请求携带的自定义header字段

服务器对预检请求做出响应, 在response中需要携带以下header : 

```
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
```

分别为 限制请求的源域, 允许请求的Method, 允许携带的自定义 header

还可以设置header字段 `Access-Control-Max-Age` , 表明该预检请求的有效时间, 该时间范围内浏览器不会为同一请求发起预检请求

预检通过会浏览器才会发送真实请求

#### [附带身份凭证的请求](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS#附带身份凭证的请求)

( 点击标题查看MDN文档 ↑↑↑ )

在需要 基于 [HTTP cookies](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies) 和 HTTP 认证信息发送身份凭证 时, 需要将 request 的 `withCredentials` 标志设为 `true` , 同时在服务端的 response 中也需要添加header:  [`Access-Control-Allow-Credentials: true`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials) 

##### ! 注意项 ! : 

> - 服务器不能将 `Access-Control-Allow-Origin` 的值设为通配符“`*`”，而应将其设置为特定的域，如：`Access-Control-Allow-Origin: https://example.com`。
> - 服务器不能将 `Access-Control-Allow-Headers` 的值设为通配符“`*`”，而应将其设置为首部名称的列表，如：`Access-Control-Allow-Headers: X-PINGOTHER, Content-Type`
> - 服务器不能将 `Access-Control-Allow-Methods` 的值设为通配符“`*`”，而应将其设置为特定请求方法名称的列表，如：`Access-Control-Allow-Methods: POST, GET`

## 解决方案

### 1. 使用flask-CORS

```python
from flask_cors import CORS

app = Flask(__name__)

CORS(app, supports_credentials=True)
```

### 2. 手写 flask 的 response 拦截器

下边是一个简单的拦截器, 可以接受使用json传输数据的http跨域请求, 具体设置为: 

- 接受来自任何域的请求
- 允许附带身份凭证
- 允许的请求方式有: GET, POST, PUT, DELETE
- 允许的自定义 header 有: content-type
- 允许的 content-type 的值有: application/json

```python
def after_req(resp):
    resp.headers["Access-Control-Allow-Origin"] = request.origin
    resp.headers["Access-Control-Allow-Credentials"] = "true"
    resp.headers["Access-Control-Allow-Methods"] = "GET, POST, PUT, DELETE"
    resp.headers["Access-Control-Allow-Headers"] = "content-type"
    resp.headers["content-type"] = "application/json"
    return resp
    
app.after_request(after_req)
```

### 3. 前端代理

通过设置 axios 的 proxy 选项配置前端的代理服务器，所有请求都发送到同域内的前端服务器，再由前端转发给后端

```javascript
const axios = require('axios');

const instance = axios.create({
    baseURL: 'http://localhost:3000/',
    proxy: {
        host: '127.0.0.1',
        port: 9000,
    },
});

instance.get('/api/users').then(response => {
    console.log(response.data);
});
```

## Reference

> - flask-github : https://github.com/pallets/flask
> - flask-document : https://flask.palletsprojects.com/en/2.1.x/
> - mdn-CORS : https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS

