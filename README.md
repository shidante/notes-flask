# notes-flask

概念篇：

1. 程序实例

	所有Flask 程序都必须创建一个程序实例。Web 服务器使用一种名为Web 服务器网关接口（Web Server Gateway Interface，WSGI）的协议，把接收自客户端的所有请求都转交给这个对象处理。程序实例是Flask 类的对象。
	
	from flask import Flask
	app = Flask(__name__)
	
	Flask 类的构造函数只有一个必须指定的参数，即程序主模块或包的名字。在大多数程序中，Python 的__name__ 变量就是所需的值。
