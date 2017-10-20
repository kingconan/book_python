## 财务计算

```py
1. 十进制浮点数的运算，由于计算机是二进制模拟的，迫于位数限制，总是逼近的。用 Decimal(str(123)) 来搞，和java的处理
方式一样
```

## Python-Redis

```py
1. 机器上先安装redis并启动

2. test redis
>> redis-cli可以进入redis的命令行，进行简单的get set测试

3. pip install redis,安装python的redis包

import redis
r = redis.Redis(host="127.0.0.1", port=6379, db=0)
r.set("k1", "hello world")
r.get("k1")
r.keys()
r.dbsize()
r.delete("k1")
r.save() #将数据写回磁盘，保存时阻塞
r.flushdb() #清空r中数据


#redis学习再看文档请 https://pypi.python.org/pypi/redis/2.9.1
```

## python装饰器

```py
1. 修改其他函数功能的函数。可以在不改变函数定义的同时，给函数加功能。OOP里面，可以通过继承时override父函数来达到
同样效果

2. 函数也是对象，所以可以将函数赋值给变量，通过变量来调用函数。函数中可以定义函数，也可以返回函数，函数可作为参数传递

3. 定义好的decorator函数，可以通过@来使用

def log(para="def"):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print "%s call %s" % (para,func.__name__)
            return func(*args, **kwargs)

        return wrapper
    return decorator


@log()
def test_decorator():
    print "normal function"

4. 常见场景
    #1 预处理/后处理，网站的授权检查，相当于中间件功能，配置上下文
    #2 记录函数行为，日志，缓存，记时
    #3 注入参数，提供默认参数，生成参数
    #4 修改调用的上下文，现场异步或者并行，类方法

5. 可以用类实现，类里可以实现更丰富的功能，而且可以通过继承扩展

from functools import wraps

class logit(object):
    def __init__(self, logfile='out.log'):
        self.logfile = logfile

    def __call__(self, func):
        @wraps(func)
        def wrapped_function(*args, **kwargs):
            log_string = func.__name__ + " was called"
            print(log_string)
            # 打开logfile并写入
            with open(self.logfile, 'a') as opened_file:
                # 现在将日志打到指定的文件
                opened_file.write(log_string + '\n')
            # 现在，发送一个通知
            self.notify()
            return func(*args, **kwargs)
        return wrapped_function

    def notify(self):
        # logit只打日志，不做别的
        pass
```

## FLASK

部署flask，Flask+Gunicorn+Gevent+Supervisor+Nginx 或者 flask + wsgi + nginx

```bash
fx_app //项目文件夹
├── fx_app //项目主文件夹
│   ├── __init__.py //模块初始化
│   ├── static //css，js等资源文件
│   │   ├── css
│   │   │   ├── bootstrap-theme.min.css
│   │   │   └── bootstrap.min.css
│   │   ├── fonts
│   │   │   ├── glyphicons-halflings-regular.eot
│   │   │   ├── glyphicons-halflings-regular.svg
│   │   │   ├── glyphicons-halflings-regular.ttf
│   │   │   ├── glyphicons-halflings-regular.woff
│   │   │   └── glyphicons-halflings-regular.woff2
│   │   ├── images
│   │   └── js
│   │       ├── bootstrap.min.js
│   │       ├── jquery.ajaxupload.js
│   │       └── jquery.js
│   ├── templates //布局相关文件
│   │   ├── index.html
│   │   ├── layout.html //base 模板文件
│   │   └── upload.html
│   ├── upload
│   │   └── Smartrade_International_Co._Ltd_MT4_PL_September_2017.xlsx
│   ├── util
│   ├── views.py
│   └── views.pyc
├── instance //私有配置文件
│   └── config.py
└── run.py //入口函数
```

```py
//run.py
# -*- coding: utf-8 -*-

from fx_app import app
app.run(debug=True)
```

```py
//fx_app/__init__.py
# -*- coding: utf-8 -*-

from flask import Flask
app = Flask(__name__)

import fx_app.views
```

```py
//fx_app/views.py
# -*- coding: utf-8 -*-

from flask import Flask, request, redirect, jsonify
from flask import render_template
from werkzeug.utils import secure_filename
import os

from . import app

UPLOAD_FOLDER = os.path.abspath('./fx_app/upload')

from excel_task import parse_fx_excel

@app.route("/cal")
def cal():
    return "Hell world"


@app.route("/")
def index():
    return render_template("index.html",name="index")


@app.route("/upload", methods=['GET', 'POST'])
def upload():
    if request.method == 'POST':
        if 'file' not in request.files:
            return redirect(request.url)
        file = request.files["file"]
        if file.filename == '':
            return redirect(request.url)

        if file:
            folder = os.path.join(UPLOAD_FOLDER)
            filename = secure_filename(file.filename)
            print folder
            filepath = os.path.join(folder, filename)
            file.save(filepath)

            data = parse_fx_excel(filepath)

            return jsonify({
                "ok": 0,
                "msg": "ok",
                "obj": data
            })

    return render_template("upload.html", name="upload")
```

```html
//layout.html
<!doctype html>
<html>
  <head>
    {% block head %}
    <link rel="stylesheet" href="{{ url_for('static',filename='css/bootstrap.min.css') }}" />
    <link rel="stylesheet" type="text/css" href="http://ajax.googleapis.com/ajax/libs/jqueryui/1.8/themes/ui-lightness/jquery-ui.css" />
      <style>
          html,body{
              height: 100%;
              padding: 0;
              margin: 0;
          }
      </style>
    <title>{% block title %}{% endblock %} - My Webpage</title>
    {% endblock %}
  </head>
  <body>
    <div id="content" style="height: 100%;width: 100%;">{% block content %}{% endblock %}</div>
    <div id="footer" style="text-align: center;background-color: #3c3c3c;padding: 60px;color:grey">
      {% block footer %}
      &copy; Copyright 2017
      {% endblock %}
    </div>

    {% block js %}
    <script type="text/javascript" src="{{ url_for('static',filename='js/jquery.js') }}"></script>
    <script type="text/javascript" src="{{ url_for('static',filename='js/bootstrap.min.js') }}"></script>
    <script type="text/javascript" src="{{ url_for('static',filename='js/jquery.ajaxupload.js') }}"></script>
    <script src="http://ajax.googleapis.com/ajax/libs/jqueryui/1.8/jquery-ui.min.js"></script>
    {% endblock %}
  </body>
</html>
```

```html
//upload.html
{% extends "layout.html" %}
{% block title %}Upload{% endblock %}
{% block head %}
  {{ super() }}
{% endblock %}
{% block content %}
<div style="padding: 30px;text-align: center;width: 600px;margin-left: auto;margin-right: auto">
    <p style="background-color: lightblue;padding: 30px;line-height: 28px;font-size: 14px">
        功能说明，啊哈哈哈，这里是功能说明
    </p>
    <div style="text-align:center;padding:20px;border:1px dashed #909090;" id="promptzone">
        click to upload
    </div>
    <div id="progressbar"></div>
    <div id="result"></div>
</div>
{% endblock %}
{% block js %}
{{ super() }}
<script>
    jQuery(function ($) {
    $.ajaxUploadSettings.name = 'file';

    $('#promptzone').ajaxUploadPrompt({
        url : 'upload',
        beforeSend : function () {
            $('#promptzone, #result').hide();
        },
        onprogress : function (e) {
            if (e.lengthComputable) {
                var percentComplete = e.loaded / e.total;
                // Show in progressbar
                $( "#progressbar" ).progressbar({
                    value: percentComplete*100,
                    complete: function () {
                        $(this).progressbar( "destroy" );
                        $("#result").show()
                        $("#result").text("上传成功，数据处理中...")
                    }
                });
            }
        },
        error : function () {
        },
        success : function (data) {
            console.log(data)
            $("#result").text("处理完成")
        }
    });
});
</script>
{% endblock %}
```



