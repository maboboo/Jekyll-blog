---
layout: post
title: python 解析文本
tags:
- Demo

categories: python
---



博客内容是在学习了[实验楼Python 文本解析器](https://www.shiyanlou.com/courses/70/labs/307/document)后的学习总结。
## Demo 功能及效果
将简单的文本解析成为一个 HTML 页面的小程序。可以以此为原型，扩展成为解析markdown 文本的解析器。

- 文本内容

```
Welcome to ShiYanLou

ShiYanLou is the first experiment with IT as the core of online education platform.*Our aim is to do the experiment, easy to learn IT*.

Course

-Basic Course

-Project Course

-Evaluation Course

Contact us

-Web:http://www.shiyanlou.com

-QQ Group:241818371

-E-mail:support@shiyanlou.com
```
- 解析成的 HTML 页面![](https://dn-anything-about-doc.qbox.me/python_markup/1.png)

## 项目代码以及注释

- 文本生成模块 util.py 

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def lines(file):
    """
    生成器，在文本最后加一空行
    """
    for line in file:
        yield line
    yield '\n' # 这一行等到上边的for循环结束以后才会执行。

def blocks(file):
    """
    生成器，生成单独的文本块
    当line.strip()为False，block不为空时，才会执行yield 说明，当文本最后一行为空行时，才能返回最后一个文本块
    """
    block = []
    for line in lines(file):
        if line.strip():  # 去掉line的 行首与行尾空格，并判断真假。 字符串为空时 ==》 False
            block.append(line) #如果为真， 将line添加到 block 中
        elif block: #如果line.strip()为假，判断block是否为空。
            yield ''.join(block).strip() #  如果block不为空，将block中的字符串连接其来，连接符为空格
            block = [] # 清除block 缓存。
```

- 文本规则模块 rules.py

```
#!/usr/vin/env python3
# -*- codiong: utf-8 -*-
import logging
logging.basicConfig(level=logging.INFO)

"""
这个模块内容是给每一种文本类型（一级标题，二级标题，列表等）定义判断规则。
每一个Rule子类都有condition方法，用来判断文本是否它符合规则。从Rule继承的action方法用来给符合的文本加标记。
"""

class Rule:
    """
    规则父类
    """

    def action(self, block, handler):
        """
        加标记
        """
        handler.start(self.type) # 标签头
        handler.feed(block) # 标签内容
        handler.end(self.type) # 标签尾
        return True

class HeadingRule(Rule):
    """
    一号标题规则
    判断文本块是否是标题
    """
    type = 'heading'
    def condition(self, block):
        """
        判断文本块是否符合规则
        """
        # 当不含有换行符，长度大于70 不以：结尾时， block为 标题。
        return not '\n' in block and len(block) <= 70 and not block[-1] == ':'


"""
TitleRule与HeadingRule 结合起来判断标题是一级标题还是二级标题。
在 markup.py 中的 BasicTextParser 中先添加 TitleRule 规则，后添加 HeadingRule 规则。
根据 TitleRule 规则可以知道，会把第一个 block 设为一级标题，后边的 block 都会调用 HeadingRule 规则。
"""
class TitleRule(HeadingRule):
    """
    二号标题规则
    判断是否是第一次出现标题。
    action 方法打印 <h1 style="color: #1ABC9C;">%s</h1>
    """
    type = 'title'
    first = True

    def condition(self, block):
        if not self.first: 
            return False
        self.first = False
        return HeadingRule.condition(self, block);

class ListItemRule(Rule):
    """
    列表项规则
    """
    type = 'listitem'
    def condition(self, block):
        return block[0] == '-'
    
    def action(self, block, handler):
        handler.start(self.type)
        handler.feed(block[1:].strip())
        handler.end(self.type)
        return True


class ListRule(ListItemRule):
    """
    列表规则
    """
    type = 'list'
    inside = False
    def condition(self, block):
        return True

    def action(self, block, handler):
        if not self.inside and ListItemRule.condition(self, block):
            handler.start(self.type)
            self.inside = True
        elif self.inside and not ListItemRule.condition(self, block):
            handler.end(self.type)
            self.inside = False
        return False

class ParagraphRule(Rule):
    """
    段落规则
    """
    type = 'paragraph'
    def condition(self, block):
        return True
```

- 处理程序模块 handlers.py

```
#!/usr/bin/env pyhon3
# -*- coding: utf-8 -*-

import logging
logging.basicConfig(level=logging.INFO)

"""
Handler类 处理程序父类，有一个回调函数 callback 可以根据传入的name参数调用相应的方法。
"""

class Handler: # handler-处理程序
    """
    处理程序父类
    """
    def callback(self, prefix, name, *args): # prefix-字首
        method = getattr(self, prefix + name, None) # getattr（x, y) 等价与 x.y  从一个对象中获得名字为y的属性。
        if callable(method):  # callable 返回某个对象是否可以被调用
            return method(*args)

    def start(self, name):
        self.callback('start_', name)


    def end(self, name):
        self.callback('end_', name)

    def sub(self, name):
        def substitution(match):
            result = self.callback('sub_', name, match)
            if result is None:
                result = match.group(0)
            return result
        return substitution

class HTMLRenderer(Handler):
    """
    HTHL 处理程序，给文本块加相应的 HTML 标记
    """
    def start_document(self): # document-文件
        print('<html><head><title>ShiYanLou</title></head><body>')

    def end_document(self):
        print('</body></html>')

    def start_paragraph(self):# parahraph-段落
        print('<p style="color: #444;">')

    def end_paragraph(self):
        print('</p>')

    def start_heading(self):
        print('<h2 style="color: #68BE5D;">')

    def end_heading(self):
        print('</h2>')

    def start_list(self):
        print('<ul style="color: #363736;">')

    def end_list(self):
        print('</ul>')

    def start_listitem(self):
        print('<li>')

    def end_listitem(self):
        print('</li>')

    def start_title(self):
        print('<h1 style="color: #1ABC9C;">')

    def end_title(self):
        print('</h1>')

    def sub_emphasis(self, match):
        return '<em>%s</em>' % match.group(1)

    def sub_url(self, match):
        return '<a target="_blank" style="text-decoration: none;color: #BC1A4B;" href="%s">%s</a>' % (match.group(1), match.group(1))

    def sub_mail(self, match):
        return '<a style="text-decoration: none; color: #BC1A4B;" href="mailto:%s">%s</a>' % (match.group(1), match.group(1))
    
    def feed(self, data):
        print(data)

```

- 主程序文本 markup.py

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import sys,re
from handlers import *
from util import *
from rules import *
import logging
logging.basicConfig(level=logging.INFO)

class Parser:
    """
    解析器父类
    """
    def __init__(self, handler):
        self.handler = handler # 处理程序对象
        self.rules = [] # 判断文本种类规则类列表
        self.filters = [] # 判断 url Email 等正则函数列表 re.sub

    def addRule(self, rule):
        """
        添加规则
        """
        self.rules.append(rule) 

    def addFilter(self, pattern, name):
        """
        添加过滤器 
        """
        def filter(block, handler):
            return re.sub(pattern, handler.sub(name), block)
        self.filters.append(filter)

    def parse(self, file):
        """
        解析
        """
        #print(file)  ---> <_io.TextIOWrapper name='<stdin>' mode='r' encoding='UTF-8'>
        self.handler.start('document')
        for block in blocks(file): # 循环文本块
            for filter in self.filters: # 
                block = filter(block, self.handler)
            for rule in self.rules:
                if rule.condition(block):
                    last = rule.action(block, self.handler)
                    if last:
                        break
        self.handler.end('document')

class BasicTextParser(Parser):
    """
    纯文本解析器
    """
    def __init__(self, handler):
        Parser.__init__(self, handler)
        self.addRule(ListRule())
        self.addRule(ListItemRule())
        self.addRule(TitleRule())
        self.addRule(HeadingRule())
        self.addRule(ParagraphRule())

        self.addFilter(r'\*(.+?)\*', 'emphasis') # 重点内容，两个 * 号之间的文本
        self.addFilter(r'(http://[\.a-zA-Z/]+)', 'url') # 提取url地址正则。
        self.addFilter(r'([\.a-zA-Z]+@[\.a-zA-Z]+[a-zA-Z]+)', 'mail') # 提取emali地址正则

"""
运行程序
"""

handler = HTMLRenderer() # 初始化处理程序，
parser = BasicTextParser(handler) # 初始化文本解析器
parser.parse(sys.stdin) # 执行解析方法
```

## 总结


