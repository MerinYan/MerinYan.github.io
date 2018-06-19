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
people/templates/people/line.html  
```html
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <!-- 引入 ECharts 文件 使用 static 关键字-->
    <script src="{% static 'people/js/echarts.common.min.js' %}"></script>
</head>
</html>
<body>
    <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
    <div id="main" style="width:800px;height:600px;"></div>
    <script type="text/javascript">
        var rx = {{ rx|safe }};
        var ry = {{ ry|safe }};
        // 基于准备好的dom，初始化echarts实例
        var myChart = echarts.init(document.getElementById('main'));
        // 指定图表的配置项和数据
        option = {
            xAxis: rx,
            yAxis: {
                type: 'value'
            },
            series: ry
        };
        // 使用刚指定的配置项和数据显示图表
        myChart.setOption(option);
    </script>
</body>
```

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