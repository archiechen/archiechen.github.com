---
layout: post
title: "使用doctest测试重载dict.sefdefault方法"
date: 2012-10-31 17:58
comments: true
categories: python
---

为了避免字典中取出的值为None，通常会使用dict的setdefult方法：
{% codeblock lang:python%}
task = dict()
urls = task.setdefult('urls',[])
{% endcodeblock %}
当字典中存在key时，无论value是否为None，setdefault方法都会忽略default值，直接返回value。为了避免这种情况，引入了一段重复代码逻辑，每次取一个key都要这样写：
{% codeblock lang:python%}
urls = task.get("urls") if task.get("urls") else []
dirs = task.get("dirs") if task.get("dirs") else []
{% endcodeblock %}
代码可读性下降了，决定采用采用子类化内建类型的方式重构。重构后代码如下：
{% codeblock lang:python%}
task = alwaysdefaultdict(task)
urls = task.setdefault('urls',[])
dirs = task.setdefault('dirs',[])
{% endcodeblock %}

重构过程中，先写doctest，alwaysdefaultdict.py文件内容如下：
{% codeblock lang:python%}
#-*- coding:utf-8 -*-
'''
重载setdefault方法，当key存在，并且value为空字符串或None时，也会设置为defaultvalue。
>>> d = alwaysdefaultdict()
>>> d.setdefault('key','')
''
>>> d.setdefault('key','default')
'default'
>>> d.get('key')
'default'
>>> d.setdefault('key','newvalue')
'default'
>>> d.get('no_key')
>>> d['newkey']='newvalue'
>>> d.get('newkey')
'newvalue'
>>> d['newkey']
'newvalue'
'''
class alwaysdefaultdict(dict):
    pass
{% endcodeblock %}
这时候执行测试的结果是：
    $ nosetests --with-doctest alwaysdefaultdict.py
    F
    ......
    Traceback (most recent call last):
    ......
    Failed example:
        d.setdefault('key','default')
    Expected:
        'default'
    Got:
        ''
    ......
    Failed example:
        d.get('key')
    Expected:
        'default'
    Got:
        ''
    ......
    Failed example:
        d.setdefault('key','newvalue')
    Expected:
        'default'
    Got:
        ''
    ----------------------------------------------------------------------
    Ran 1 test in 0.035s

    FAILED (failures=1)
然后实现代码逻辑，最后代码如下：
{% gist 3986136 %}
执行doctest的结果：
    $ nosetests --with-doctest alwaysdefaultdict.py
    .
    ----------------------------------------------------------------------
    Ran 1 test in 0.096s

    OK
