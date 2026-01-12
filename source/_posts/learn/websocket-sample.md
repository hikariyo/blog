---
title: WebSocket简单应用
date: 2022-05-24 18:46:46.0
updated: 2022-12-10 19:38:29.399
categories: 
- Learn
tags: 
- JS
- Node
---

## 简介

如果想要实现实时与服务器连接，一个简单的方法如下：
```js
setInterval(() => {
    ajax()
}, 5000)
```
通过这种方式可以达到不断刷新的目的，但是它存在着诸多弊端：

+ 如果没有新的内容的话，反复请求同一内容浪费流量。
+ 如果有新的内容，用户无法第一时间得到，需要等待下一次定时器被调用。

所有出现了`WebSocket`这种技术，它可以实现服务器和客户端双向通信，不仅服务器实时和客户端发送消息，客户端也向服务器实时发送消息，没有多余的请求。

## NodeJS环境

安装`nodejs-websocket`，运行：

```bash
yarn add nodejs-websocket
```

之后写这些代码即可：

```js
const ws = require('nodejs-websocket')

let count = 0

const server = ws.createServer(conn => {
    count++

    conn.name = 'User ' + count

    broadcast(`${conn.name} joined`)

    conn.on('text', msg => {
        broadcast(`${conn.name}: ${msg}`)
    })

    conn.on('close', () => {
        broadcast(conn.name + ' left')
    })

    conn.on('error', () => {
        console.log('Something went wrong')
    })
})

/**
 * 遍历所有连接，发送指定消息
 */
function broadcast(msg) {
    server.connections.forEach(conn => {
        conn.send(msg)
    })
}

server.listen(8080, () => {
    console.log('Server is running on port 8080')
})
```

不是很难理解，就是监听了几个事件，并作出对应的处理，不多解释了。

## 浏览器环境

首先准备三个组件：

```html
<input id="message" type="text" placeholder="Message">
<button id="button">Send</button>
<div id="content"></div>
```

之后这些JS代码：

```js
const socket = new WebSocket('ws://localhost:8080')
const content = document.getElementById('content')
const message = document.getElementById('message')
const button = document.getElementById('button')

socket.onopen = () => {
    console.log('Connection opened')
}

socket.onmessage = data => {
    console.log(data)
    addMessage(data.data)
}

socket.onclose = () => {
    addMessage('Connection closed')
}

socket.onerror = () => {
    addMessage('Something went wrong')
}

function addMessage(msg) {
    const div = document.createElement('div')
    div.innerText = msg
    content.appendChild(div)
}

button.onclick = () => {
    socket.send(message.value)
}
```
