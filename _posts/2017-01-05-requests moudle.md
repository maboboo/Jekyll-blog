---
layout: post
title: 模块学习笔记 requests
categories: python
tag:
- 网络

---

requests模块
通过学习 [requests 官方文档](http://docs.python-requests.org/en/latest/user/quickstart.html),整理的笔记.

## request 简单入门
### 1. 发送 http 请求.


```
#get类型
r = requests.get('https://api.github.com/events')
#post类型
r = r = requests.post('http://httpbin.org/post', data = {'key':'value'})
#put类型
r = requests.put('http://httpbin.org/put', data = {'key':'value'})
#delete类型
r = requests.delete('http://httpbin.org/delete')
#head类型
 r = requests.head('http://httpbin.org/get')
#options类型
r = requests.options('http://httpbin.org/get')
```
### 2.响应结果信息

```
# r 类型为 <class 'requests.models.Response'>

# 获取响应码
r.status_code

# 响应内容
r.content  # 以字节方式获取响应内容
r.text  # 以文本方式获取响应内容

# 响应头
print(r.headers)
print(r.headers['Content-Type'])
print(r.headers.get('content-type'))

# 获取/修改网页编码
>>> r = requests.get('https://www.baidu.com')
>>> r.encoding
'ISO-8859-1'
>>> r.encoding = 'utf-8'
>>> r.encoding
'utf-8'
```
- json 格式响应内容

```
import requests

r = requests.get('https://api.github.com/events')
r.json()  #  返回一个list 或者 dict
```

- 图片

```
#	coding=utf-8
import	requests
r =	requests.get("https://github.com/favicon.ico")
i = Image.open(BytesIO(r.content))
print(type(i))  # <class 'PIL.IcoImagePlugin.IcoImageFile'>

# 保存为文件
with open('favicon.ico', 'wb') as f:
    f.write(r.content)
    f.close()
```



