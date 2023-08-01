---
title: Ajax
date: 2023-08-01 14:01:03
tags: Ajax
categories: Front-end
---

# Ajax

* 概念：Asynchronous JavaScript And XML，异步的JavaScripti和XML
* 作用：
  * **数据交换**：通过Ajx可以给服务器**发送请求**，并**获取服务器响应的数据**。
  * **异步交互**：可以在**不重新加载整个页面**的情况下，与服务器**交换数据并更新部分网页**的技术，如：搜索联想、用户名是否可用的校验等等。

![image-20230801140413775](./Ajax/image-20230801140413775.png)



# 原生Ajax

步骤：

* 创建XMLHttpRequest对象：用于和服务器交换数据
* 向服务器发送请求
* 获取服务器响应数据



示例：

```html
<!DOCTYPE html>
<html>
<body>

<div id="demo">
<h2>The XMLHttpRequest Object</h2>
<button type="button" onclick="loadDoc()">Change Content</button>
</div>

<script>
function loadDoc() {
    //创建XMLHttpRequest对象
    const xhttp = new XMLHttpRequest();
    //监听数据
    xhttp.onload = function() {
        document.getElementById("demo").innerHTML =
        this.responseText; //以字符串的形式返回响应数据
    }
    //向服务器发送请求
    xhttp.open("GET", "https://www.w3schools.com/js/ajax_info.txt");
    xhttp.send();
}
</script>

</body>
</html>
```



## XMLHttpRequest 对象属性

| Property           | Description                                                  |
| :----------------- | :----------------------------------------------------------- |
| onload             | 定义接收(加载)请求时要调用的函数。                           |
| onreadystatechange | 定义当readyState 属性发生变化时被调用的函数                  |
| readyState         | 保存 XMLHttpRequest 的状态 <br />* 0: 请求未初始化 <br />* 1: 服务器链接已建立 <br />* 2: 请求已收到  <br />* 3: 正在处理请求 <br />* 4: 请求已完成且相应已建立 |
| responseText       | Returns the response data as a string                        |
| responseXML        | Returns the response data as XML data                        |
| status             | 返回请求的状态号 <br />* 200: "OK" <br />* 403: "Forbidden" <br />* 404: "Not Found" <br /><br />完整列表请访问 [Http Messages Reference](https://www.w3schools.com/tags/ref_httpmessages.asp) |
| statusText         | 返回状态文本 (e.g. "OK" or "Not Found")                      |



所以为了确保请求成功

可以将代码写成这样

```js
function loadDoc() {
    //创建XMLHttpRequest对象
    const xhttp = new XMLHttpRequest();
    //监听数据
    xhttp.onload = function() {
        if (this.readyState == 4 && this.status == 200) { //确保请求成功
            document.getElementById("demo").innerHTML =
            this.responseText; //以字符串的形式返回响应数据
        }
    }
    //向服务器发送请求
    xhttp.open("GET", "https://www.w3schools.com/js/ajax_info.txt");
    xhttp.send();
}
```





# Axios

原生Ajax比较繁琐，以及存在一些早期的浏览器兼容问题， 现在项目中一般使用原生Ajax封装的Axios

* 介绍：Axios 对原生Ajax进行了封装，简化了书写，加快开发
* 官网：https://www.axios-http.cn/



## Axios入门

1. 引入 Axios.js 文件

2. 使用Axios发送请求，获取响应结果

示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <title>Document</title>
</head>
<body>
    <input type="button" value="获取数据get" onclick="get()">
    <input type="button" value="获取数据post" onclick="post()">
    
</body>
<script>
    function get(){
        //通过Axios发送异步请求-get
        axios({
            method: 'get',
            url:'https://autumnfish.cn/api/joke'
        }).then( result =>{
            console.log(result.data);
        })
    }

    function post(){
        //通过Axios发送异步请求-post
        axios({
            method: 'post',
            url:'https://autumnfish.cn/api/user/check',
            data: "username=name"
        }).then( result =>{
            console.log(result.data);
        })
    }
</script>

</html>
```

![image-20230801145126573](./Ajax/image-20230801145126573.png)





## Axios请求方式别名

更加快速的使用

别名：

* axios.get(url [config])
* axios.delete(url [config])
* axios.post(url [data[,config]])
* axios.put(url [data[,config]])

> 发送get请求：

```js
axios.get("https://autumnfish.cn/api/joke").then((result)=>{
	console.log(result.data)	
})
```

> 发送post请求

```js
axios.post("https://autumnfish.cn/api/user/check","username=name").then((result)=>{
	console.log(result.data)	
})
```



简化代码之后：

```html
<script>
    function get(){
        //通过Axios发送异步请求-get
        axios.get("https://autumnfish.cn/api/joke").then((result)=>{
	console.log(result.data)	
})
    }

    function post(){
        //通过Axios发送异步请求-post
        axios.post("https://autumnfish.cn/api/user/check","username=name").then((result)=>{
	console.log(result.data)	
})
    }
</script>
```



