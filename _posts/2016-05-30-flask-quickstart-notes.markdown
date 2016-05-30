---
layout: post
title:  "Flask学习笔记"
date:   2016-05-30 14:44:00 +0800
categories: python
tags: Python
---

## 一. 安装

输入`pip install flask`开始安装

![Alt text](http://7xp9tm.com1.z0.glb.clouddn.com/20160503-01.png)

Flask依赖了4个包

* Werkzeug
* Jinja2
* itsdangerous
* MarkupSafe

都不大，总共才1M多
<!--more-->

## 二. 简单例子

新建一个文件hello.py，内容如下，随便放哪个目录下：

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

然后执行脚本

    python hello.py

默认会绑定5000端口

![Alt text](http://7xp9tm.com1.z0.glb.clouddn.com/20160503-02.png)

访问[http://127.0.0.1:5000/](http://127.0.0.1:5000/)，效果如下：

![Alt text](http://7xp9tm.com1.z0.glb.clouddn.com/20160503-03.png)

如果想以DEBUG模式启动服务，那么就在app.run()的参数中加上debug=True，同理，还能指定绑定IP以及端口：

```python
if __name__ == "__main__":
    app.run(host='0.0.0.0',port=5001,debug=True)
```

## 三. 路由规则

刚才的代码中

    @app.route("/")

这一行，说明了访问[http://127.0.0.1:5000/](http://127.0.0.1:5000/)的时候调用`hello()`。
一般来说，这个URL应该指向index页面，所以我们有

```python
@app.route('/')
def index():
    return 'Index Page'
@app.route('/hello')
def hello():
    return 'Hello World'
```

这样一来，访问[http://127.0.0.1:5000/](http://127.0.0.1:5000/)的时候调用index()，
而访问[http://127.0.0.1:5000/hello](http://127.0.0.1:5000/hello)的时候调用hello()。

`@app.route()`还有别的参数，比如：

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
      do_the_login()
    else:
      show_the_login_form()
```

用methods指定了只有GET和POST可以访问，并且可以用`request.method`获取到请求方法

除了最常见的GET、POST以外，经常用于RESTful的方法还有PUT和DELETE，除了这些之外，Flack还支持HEAD、OPTIONS等方法。

## 四. 获取url

可以通过`url_for`得到某个view的url（通过`@app.route( )`获取），或者静态文件的url，跟Django一样，静态文件放在`<应用目录>/static`下，当然，Django可以改就是了。

## 五. 渲染模板

模板放在`<应用目录>/templates`下。

模板引擎用的是Jinja2，模板语言跟Django模板引擎或者PHP的Twig有点像。
先写个模板`hello.html`，内容如下：

```html
<!doctype html>
<title>Hello from Flask</title>
{% if name %}
  <h1>Hello {{ name }}!</h1>
{% else %}
  <h1>Hello World!</h1>
{% endif %}
```

然后写Python脚本：

```python
from flask import render_template
from flask import Flask
app = Flask(__name__)

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)

if __name__ == "__main__":
    app.run()
```

主要还是通过`render_template`函数来渲染页面，第一个参数是模板文件名，接下去是传给模板的参数

## 六. 获取请求参数

#### GET与POST请求参数

关于获取POST请求的参数的例子如下：

```python
from flask import request
from flask import Flask
from flask import render_template
app = Flask(__name__)

@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    # the code below is executed if the request method
    # was GET or the credentials were invalid
    return render_template('login.html', error=error)

if __name__ == "__main__":
    app.run(host='127.0.0.1',port=5000,debug=True)
```

#### 关于文件上传

文件上传时，会把文件存放在request.files这个dict下，可以通过form的参数名获取文件。再使用save方法将数据保存在服务器的磁盘上：

```python
@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
```

也许我们不愿意让用户知道上传上来的文件叫什么名字，我们可以使用werkzeug中的secure_filename来加密文件名：

```python
from flask import request
from werkzeug import secure_filename
@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/' + secure_filename(f.filename))
```

#### Cookies

Cookie可以通过`request.cookies.get('key')`来获取
设置Cookie则是在response报文中设置，例如：

```python
@app.route('/')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp
```

#### Session

首先是有个session字典，用`session['key']`获取参数，用`session.pop('key', 'value')`设置参数，用`if 'key' in session`检查session中是否有这个参数。

这里有个Session的例子：

```python
from flask import Flask, session, redirect, url_for, escape, request

app = Flask(__name__)

@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form action="" method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))

# set the secret key.  keep this really secret:
app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'
```

#### 页面重定向

以下是使用redirect重定向页面，以及使用abort报错的例子：

```python
from flask import abort, redirect, url_for
@app.route('/')
def index():
    return redirect(url_for('login'))
@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
```

错误页面也可以自己定义：

```python
from flask import render_template
@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```
