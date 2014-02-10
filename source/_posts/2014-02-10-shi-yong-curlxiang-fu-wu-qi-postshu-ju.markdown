---
layout: post
title: "使用CURL向服务器POST数据"
date: 2014-02-10 11:59
comments: true
categories: 
---

1. 向服务器POST JSON数据
    ```
    curl -X POST -H "Content-Type: application/json" -d '{"username":"user","password":"password"}' http://localhost:5000/login
    ```

1. 保存cookie到文件中
    ```
    curl -X POST -H "Content-Type: application/json" -d '{"username":"user","password":"password"}' -c cookie.txt http://localhost:5000/login
    ```

1. 向服务器发请求时带cookie
    ```
    curl -X POST -H "Content-Type: application/json" -d '{"message":"Hello!"}' -b cookie.txt http://localhost:5000/messages
    ```
