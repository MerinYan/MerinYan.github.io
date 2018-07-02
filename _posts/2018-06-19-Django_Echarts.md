---
layout: default
title:  "Django Echarts"
date:   2018-06-19 10:45:52 +0800
---


![Branching]({{site.url}}/assets/images/2018-06-19-Django-Echarts.png)

*** 引用Echarts js  
下载js：echarts.common.min.js  
添加到应用中：people/static/people/js/echarts.common.min.js


*** 修改views.py  
```python
def getbar(request):
    x = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    xr = {
        'type': 'category',
        'data': x
    }
    y1 = [820 , 932 , 901 , 934 , 1290 , 1330 , 1320]
    y2 = [80 , 92 , 91 , 94 , 290 , 130 , 132]
    yr1 = {
        'type': 'line',
        'data': y1,
        'smooth': True
    }
    yr2 = {
        'type': 'line' ,
        'data': y2 ,
        'smooth': True
    }
    dict_from_html = {
        'rx': json.dumps(xr),
        'ry': json.dumps([yr1,yr2]),
    }
    return render(request, 'people/line.html', dict_from_html)
```

*** 创建模板文件  
 
[people/templates/people/line.html ]({{site.url}}/assets/file/2018-07-02-line.txt)  


整体目录结构
```
(env27) MBookPro:mysite mervin$ tree
.
├── manage.py
├── mysite
│   ├── __init__.py
│   ├── __init__.pyc
│   ├── settings.py
│   ├── settings.pyc
│   ├── urls.py
│   ├── urls.pyc
│   ├── wsgi.py
│   └── wsgi.pyc
├── people
│   ├── __init__.py
│   ├── __init__.pyc
│   ├── admin.py
│   ├── admin.pyc
│   ├── apps.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   ├── 0001_initial.pyc
│   │   ├── 0002_article.py
│   │   ├── 0002_article.pyc
│   │   ├── __init__.py
│   │   └── __init__.pyc
│   ├── models.py
│   ├── models.pyc
│   ├── static
│   │   └── people
│   │       └── js
│   │           └── echarts.common.min.js
│   ├── templates
│   │   └── people
│   │       ├── index.html
│   │       ├── line.html
│   │       └── pyecharts.html
│   ├── tests.py
│   ├── views.py
│   └── views.pyc
└── render.html
```