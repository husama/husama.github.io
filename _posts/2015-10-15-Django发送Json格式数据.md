---
layout: post_layout
title: Django发送Json格式数据
time: 2015年10月15日
location: 长春
pulished: true
excerpt_separator: "##"
---

## 简要


RESTful API是目前比较成熟的一套互联网应用程序的API设计理论.具体设计可以看一下 [RESTful API 设计指南 by 阮一峰](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

## Django的应用
本文讨论RESTful API的一个小应用,当Django作为类似Android,IPhone等APP的后台框架时,往往Response的不是一个Html,而是一些序列化的数据,比如**Json**.
所以我们需要把models里的数据提取出来转化为json,虽然有基于Django的[Django REST framework](http://www.django-rest-framework.org/)可以直接用,但是自己写其实也挺简单的:-)

## 简单的一个例子
这里我用的是python3.4和Django1.8.
### models.py
```
from django.db import models
from datetime import datetime
class Article(models.Model):
        title = models.CharField(u'标题',max_length=256)
        content = models.TextField(u'内容')
        time = models.DateTimeField(u'发表时间', auto_now_add=True)
        def __str__(self):
                return self.title
        def toDict(self):
                return {u'标题':self.title,u'内容':self.content,u'发表时间':self.time.strftime('%Y-%m-%d %H:%M:%S')}
```

### views.py
```
from django.http import HttpResponse
import json  
from article.models import Article
def toDicts(objs):
        obj_arr=[]
        for o in objs:
                obj_arr.append(o.toDict())
        return obj_arr          

def jsonAll(request):
        all_objs=Article.objects.all()
        all_dicts=toDicts(all_objs)
        all_jsons=json.dumps(all_dicts,ensure_ascii=False)  
        return HttpResponse(all_jsons)
```
models.py中的toDict将model对象格式化成字典类型,views.py中的toDicts函数将返回字典列表.
而json.dumps序列化的时候,加入ensure_ascii=False是为了防止编码错误,就像下面:

> [{“\u5185\u5bb9”: “\u6211\u7684\u7b2c\u4e00\u4e2adjango project,\u8fd4\u56deJSON\u6570\u636e”, “\u6807\u9898”: “\u80e1\u6e58\u94ed\u7684\u6587\u7ae0″,”\u53d1\u8868\u65f6\u95f4”: “2015-10-15 10:15:41”}, {“\u5185\u5bb9”: “\u53d1\u9001\u4e86\u4e00\u7ec4json\u6570\u636e”, “\u6807\u9898”: “hello,json”,”\u53d1\u8868\u65f6\u95f4″: “2015-10-15 10:14:50”}

添加ensure_ascii=False后就正常了:

> [{“标题”: “胡湘铭的文章”, “内容”: “我的第一个django project,返回JSON数据”,”发表时间”: “2015-10-15 10:15:41”}, {“标题”: “hello,json”, “内容”: “发送了一组json数据”,”发表时间”: “2015-10-15 10:14:50”}]

##Python还是很强大的:-)
