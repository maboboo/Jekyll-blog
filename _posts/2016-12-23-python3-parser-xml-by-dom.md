---
layout: post
title: python3 使用 dom 解析 xml 文件
categories: python
---

前几天公司项目需要处理 apk 中有关string资源里格式化字符串内容，android 中字符串资源都储存在 strings.xml 中，而 string.xml 文件格式是 xml ，如果使用 python 处理的话，需要解析 xml 文件。之前没有解析过 xml 文件。啊，只好现学了。常用解析 xml 文件的方法有 DOM、SAX、StAX 等。今天就要DOM解析一下

## 什么是 xml？


- XML 指可扩展标记语言（EXtensible Markup Language）
- XML 是一种标记语言，很类似 HTML
- XML 的设计宗旨是传输数据，而非显示数据
- XML 标签没有被预定义。您需要自行定义标签。
- XML 被设计为具有自我描述性。
- XML 是 W3C 的推荐标准

通俗来讲： xml 就是规定了通过文本存储信息的一种格式，方便与不同的软件直接，服务器客户端等之间用于传输信息。

下边这样的文本就是 xml

```
<?xml version="1.0" encoding="utf-8"?>
<catalog>
    <maxid>4</maxid>
    <login username="pytest" passwd='123456'>
        <caption>Python</caption>
        <item id="4">
            <caption>测试</caption>
        </item>
    </login>
    <item id="2">
        <caption>Zope</caption>
    </item>
</catalog>
```

wiki 上关于 xml 的解释 [https://en.wikipedia.org/wiki/XML ](https://en.wikipedia.org/wiki/XML)

w3school上xml 的教程 [http://www.w3school.com.cn/xml/xml_intro.asp](http://www.w3school.com.cn/xml/xml_intro.asp)


## python3 解析 xml

###
```
import xml.dom.minidom

# 打开 test.xml 文件
dom = xml.dom.minidom.parse('test.xml')
print(type(dom))
# 获得 xml 文档元素对象 
root = dom.documentElement
# 打印 node 的名称
print(root.nodeName)
print(root.nodeType)
print(root.nodeValue)
print(root.ELEMENT_NODE)

```
运行结果如下所示，可看到 dom 类型为 Document，root 类型为 Element。
```
<class 'xml.dom.minidom.Document'>
<class 'xml.dom.minidom.Element'>
catalog
1
None
1
```
如果想要得到 catalog 的子标签 maxid， 可以通过下面的方法：
```
bb = root.getElementsByTagName('maxid')
print('bb 的 type: ', type(bb))
print(bb)
b = bb[0]
print('b 的 type：', type(b))
print(b)
print(b.nodeName)
print(b.nodeType)
```
输出结果入下，我们可以看到 bb 的类型为 list， b 是其中的一个元素，类型是 Element。
```
bb 的 type:  <class 'xml.dom.minicompat.NodeList'>
[<DOM Element: maxid at 0x7f8980848af8>]
b 的 type： <class 'xml.dom.minidom.Element'>
<DOM Element: maxid at 0x7f8980848af8>
maxid
1
```
通过 getElementsByTagName 方法得到元素集合，例如我们获得 item 子标签：
```
items = root.getElementsByTagName('item')
print(items)
item1 = items[0]
print(item1.nodeName)
print(item1.nodeType)
item2 = items[1]
print(item1.nodeName)
print(item1.nodeType)
```
输出结果如下，我们可以看到通过 getElementsByTagName('item') 方法得到了 xml 文件中的两个 item 标签的集合。
```
[<DOM Element: item at 0x7fd6816ff340>, <DOM Element: item at 0x7fd6816ff470>]
item
1
item
1

```

### 得到元素的属性（id）

items 列表中存储的两个元素，都有属性 id 我们打印它们的 id 值看一看。可以通过 getAttribute（'id'）。

```
item1_id = item1.getAttribute('id')
item2_id = item2.getAttribute('id')

item1 id: 4
item2 id: 2
```
### 获得标签之间的值

例如要获得 caption 标签间的数据：

```
captions = root.getElementsByTagName('caption')
print(captions)
for caption in captions:
    data = caption.firstChild.data
```
结果如下：

```
[<DOM Element: caption at 0x7f922c49d2a8>, <DOM Element: caption at 0x7f922c49d3d8>, <DOM Element: caption at 0x7f922c49d508>]
Python
测试
Zope
```
通过 caption.firstChild.data 可以得到标签内的数据


本文主要参考自虫师的博客[http://www.cnblogs.com/fnng/p/3581433.html](http://www.cnblogs.com/fnng/p/3581433.html)


xml.dom.minidom 模块 
- [官方文档地址](https://docs.python.org/3/library/xml.dom.minidom.html)https://docs.python.org/3/library/xml.dom.minidom.html
- [中文翻译地址](http://python.usyiyi.cn/translate/python_352/library/xml.dom.minidom.html
)http://python.usyiyi.cn/translate/python_352/library/xml.dom.minidom.html


