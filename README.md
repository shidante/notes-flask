# notes-flask

#概念篇：

- 01. **程序实例**

所有Flask 程序都必须创建一个程序实例。Web 服务器使用一种名为Web 服务器网关接口（Web Server Gateway Interface，WSGI）的协议，把接收自客户端的所有请求都转交给这个对象处理。程序实例是Flask 类的对象。
``` 
from flask import Flask
app = Flask(__name__)
```
Flask 类的构造函数只有一个必须指定的参数，即程序主模块或包的名字。在大多数程序中，Python 的`__name__`变量就是所需的值。

- 02. **路由**

指URL到Python函数的映射关系。处理URL和函数之间关系的程序称为路由。
在Flask 程序中定义路由的最简便方式，是使用程序实例提供的app.route 修饰器，把修饰的函数注册为路由。下面的例子说明了如何使用这个修饰器声明路由：
```
@app.route('/')
def index():
	return '<h1>Hello World!</h1>'
```
修饰器是Python 语言的标准特性，可以使用不同的方式修改函数的行为。惯常用法是使用修饰器把函数注册为事件的处理程序。

```
@app.route('/user/<name>')
def user(name):
	return '<h1>Hello, %s</h1>' % name
```
在需要为为修饰器增加动态URL映射的情形中，只需要用`<>`标识即可，之后所有能够匹配静态部分URL的都会映射到该路由，调用视图函数，同时flask会将动态部分作为参数传入函数。

路由中的动态部分默认使用字符串，不过也可使用类型定义。例如，路由`/user/<int:id>`只会匹配动态片段`id`为整数的URL。Flask 支持在路由中使用`int`、`float`和`path`类型。`path`类型也是字符串，但不把斜线视作分隔符，而将其当作动态片段的一部分

- 03. **响应**

上例中函数返回的值称作响应，是客户端收到的内容。根据客户端的类型，响应的内容也是不同的，如果是web浏览器，则可以是显示给用户的文档。
```
视图函数响应400状态码
@app.route('/')
def index():
	return '<h1>Bad Request!</h1>', 400
```

Flask 视图函数还可以返回Response对象。make_response() 函数可接受1个、2个或3个参数（和视图函数的返回值一样），并返回一个Response 对象。
```
from flask import make_response

@app.route('/')
def index():
	response = make_response('<h1>This document carries a cookies</h1>')
	response.set_cookie('answer', '42')
	return response
```

有一种名为重定向的特殊响应类型。这种响应没有页面文档，只告诉浏览器一个新地址用以加载新页面。Flask提供了`redirect()`辅助函数，用于生成这种响应：
```
from flask import redirect
@app.route('/')
def index():
	return redirect('http://www.example.com')
```

还有一种特殊的响应由abort 函数生成，用于处理错误。
```
from flask import abort
@app.route('/user/<id>')
def get_user(id):
	user = load_user(id)
	if not user:
		abort(404)
	return '<h1>Hello, %s</h1>' % user.name
```
注意，abort 不会把控制权交还给调用它的函数，而是抛出异常把控制权交给Web服务器。


- 04. **视图函数**

上例中的函数称之为视图函数，用于返回字符串或者表单。

- 05. **程序/请求上下文(context)**

Flask 从客户端收到请求时，要让视图函数能访问一些对象，这样才能处理请求。请求对象就是一个很好的例子，它封装了客户端发送的HTTP 请求。

在多线程服务器中，多个线程同时处理不同客户端发送的不同请求时，每个线程看到的request 对象必然不同。Flask使用上下文让特定的变量在一个线程中全局可访问，与此同时却不会干扰其他线程。
```
@app.route('/')
def index():
	user_agent = request.header.get('User_Agent')
	return '<p>Your browser is %s</p>' % user_agent
```
```
上下文分为程序上下文及请求上下文

  变量名            上下文                                 说明
current_app       程序上下文        当前激活程序的程序实例
g                 程序上下文        处理请求时用作临时存储的对象。每次请求都会重设这个变量
request           请求上下文        请求对象，封装了客户端发出的HTTP 请求中的内容
session           请求上下文        用户会话，用于存储请求之间需要“记住”的值的词典
```
如果使用这些变量时我们没有激活程序上下文或请求上下文，就会导致错误。

- 06. **请求钩子(hooks)**

在处理请求之前或之后执行代码，请求钩子使用修饰器实现。Flask 支持以下4 种钩子。
```
• before_first_request:  Register a function to run before the first request is handled.
• before_request:        Register a function to run before each request.
• after_request:         Register a function to run after each request, if no unhandled exceptions occurred.
• teardown_request:      Register a function to run after each request, even if unhandled exceptions occurred.
```

- 07. **模板**

模板是一个包含响应文本的文件，其中包含用占位变量表示的动态部分，其具体值只在请求的上下文中才能知道。使用真实值替换变量，再返回最终得到的响应字符串，这一过程称为渲染。下文为示例：

```
# templates/index.html
<h1>Hello World!</h1>
```

```
# templates/user.html
<h1>Hello, {{ name }}!</h1>
```

```
# main.py
from flask import Flask, render_template
# ...
@app.route('/')
def index():
return render_template('index.html')
@app.route('/user/<name>')
def user(name):
return render_template('user.html', name=name)
```
Flask 提供的`render_template`函数把`Jinja2`模板引擎集成到了程序中。`render_template`函数的第一个参数是模板的文件名。随后的参数都是键值对，表示模板中变量对应的真实值。在这段代码中，第二个模板收到一个名为`name`的变量。

前例中的`name=name`是关键字参数，左边的“name”表示参数名，就是模板中使用的占位符；右边的“name”是当前作用域中的变量，表示同名参数的值。

**模板的变量**

在模板中使用的`{{ name }}`结构表示一个变量，它是一种特殊的占位符，告诉模板引擎这个位置的值从渲染模板时使用的数据中获取。

Jinja2 能识别所有类型的变量，甚至是一些复杂的类型，例如列表、字典和对象。在模板中使用变量的一些示例如下：
```
<p>A value from a dictionary: {{ mydict['key'] }}.</p>
<p>A value from a list: {{ mylist[3] }}.</p>
<p>A value from a list, with a variable index: {{ mylist[myintvar] }}.</p>
<p>A value from an object's method: {{ myobj.somemethod() }}.</p>
```

可以使用过滤器修改变量，过滤器名添加在变量名之后，中间使用竖线分隔。例如，下述模板以首字母大写形式显示变量name 的值：
`Hello, {{ name|capitalize }}`

**Jinja2变量过滤器**

```
过滤器名                说明
safe                渲染值时不转义
capitalize          把值的首字母转换成大写，其他字母转换成小写
lower               把值转换成小写形式
upper               把值转换成大写形式
title               把值中每个单词的首字母都转换成大写
trim                把值的首尾空格去掉
striptags           渲染之前把值中所有的HTML 标签都删掉
```

**完整的过滤器列表可在[Jinja2文档](http://jinja.pocoo.org/docs/templates/#builtin-filters)中查看。**

**模板的控制结构**

```
# 条件控制语句
{% if user %}
	Hello,{{ user }}!
{% else %}
	Hello, stranger!
{% endif %}
```

```
# 循环语句
<ul>
	{% for comment in comments %}
		<li>{{ comment }}/li>
	{% endfor %}
</ul>
```

```
# 宏语句，类似于方法/函数
{% macro render_comment(comment) %}
	<li>{{ comment }}</li>
{% endmacro %}

<ul>
	{% for comment in comments %}
		{{ render_comment(comment) }}
	{% endfor %}
</ul>
```

```
# 宏可以保存在单独的文件中，并在需要的时候导入
{% import 'macros.html' as macros %}
<ul>
	{% for comment in comments %}
		{{ marcros.render_template(comment) }}
	{% endfor %}
</ul>
```

```
#需要在多处重复使用的模板代码片段可以写入单独的文件，再包含在所有模板中，以避免重复：
{% include 'common.html' %}
```

**模板继承**

```
# 模板继承中的基模板,base.html
<html>
<head>
	{% block head %}
	<title>{% block title %}{% endblock %} -My Application</title>
	{% endblock %}
</head>
<body>
	{% block body %}
	{% endblock %}
</body>
</html>
```

```
# 模板继承中的衍生模板
{% extends "base.html" %}
{% block title %}Index{% endblock %}
{% block head %}
	 # 在基模板中其内容不是空的，所以使用super() 获取原来的内容。
	{{ super() }}
	<style>
	</style>
{% endblock %}
{% block body %}
<h1>Hello World!</h1>
{% endblock %}
```

- 08. 自定义错误页面

Flask允许程序使用基于模板的自定义错误页面。
```
@app.errorhandler(404)
def page_not_found(e):
	return render_template('404.html'), 404

@app.errorhandler(500)
def internal_server_error(e):
	return render_template('500.html'), 500
```

- 09. 链接

用于在模板中辅助构建动态的URL链接，有以下三种形式：
```
# 以视图函数名作为参数
url_for('index')
>>> /  # 相对地址

url_for('index', _external=True)
>>> http://localhost:5000/  # 绝对地址

# 以动态路由中的参数作为关键字参数传入
url_for('user', name='john', _external=True)
>>> http://localhost:5000/user/john

# 不以动态路由中的参数传入，而是额外的参数
url_for('index', page=2)
>>> /?page=2
```
生成连接程序内不同路由的链接时，使用相对地址就足够了。如果要生成在浏览器之外使用的链接，则必须使用绝对地址，例如在电子邮件中发送的链接。

- 10. 静态文件

默认设置下，Flask程序有一个名为static的特殊路由，用于引用静态文件，即/static/<filename>,例如：
```
url_for('static', filename="css/styles.css", _external=True)
>>> http://localhost:5000/static/css/style.css
```


- 11. Flash消息

请求完成后，有时需要让用户知道状态发生了变化。这里可以使用确认消息、警告或者错误提醒，`flash()`函数可实现这种效果。
```
# hello.py

from flask import Flask, render_template, session, redirect, url_for,flash
@app.route('/', methods=['GET','POST'])
def index():
	form = NameForm()
	if form.validate_on_submit():
		old_name = session.get('name')
		if old_name is not None and old_name != form.name.data:
			flash('Looks like you have changed your name!')
		session['name'] = form.name.data
		return redirect(url_for('index'))
	return render_template('index.html', form=form, name=session.get('name'))
```
仅调用`flash()`函数并不能把消息显示出来，程序使用的模板要渲染这些消息。最好在基模板中渲染Flash消息，因为这样所有页面都能使用这些消息。Flask把`get_flashed_messages()`函数开放给模板，用来获取并渲染消息。
```
# base.html
{% block content %}
<div class="container">
	{% for message in get_flashed_messages() %}
	<div class="alert alert-warning">
		<button type="button" class="close" data-dismiss="alert">&times;</button>  # &times; 是 X 关闭按钮的转义符
		{{ message }}
	</div>
	{% endfor %}
	{% block page_content %}{% endblock %}
</div>
```
在模板中使用循环是因为在之前的请求循环中每次调用`flash()`函数时都会生成一个消息，所以可能有多个消息在排队等待显示。`get_flashed_messages()`函数获取的消息在下次调用时不会再次返回，因此Flash消息只显示一次，然后就消失了。


















