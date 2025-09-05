---
title: Flask入门
date: 2025-09-04 22:56:48
tags: Python
categories: Python学习
---

学习于https://zhuanlan.zhihu.com/p/1932065916251791958

# 什么是Flask

### 1. 来源

Flask 是一个用 Python 编写的轻量级 Web 框架，名字来源于小酒瓶（flask).

**核心设计理念：**保持最小核心功能，把扩展交给开发者选择

<!--more-->

### 2. 基础介绍

**最小核心功能：**

* 请求对象和响应对象(request and response)，各个开发语言好像一样，Java的类名巨长，Python就很易懂
* URL路由：类似于Java的@GetMapping("/route")
* [Jinja2](https://zhida.zhihu.com/search?content_id=261000129&content_type=Article&match_order=1&q=Jinja2&zhida_source=entity) 模板引擎支持
* WSGI兼容
* 日志、错误处理基础支持：类似于java的log日志，try-catch异常处理机制

其他功能（如数据库 ORM、认证系统、表单校验、文件上传、Admin 管理后台等），需要我们自己实现。



安装：

```
pip install flask
```

python 3安装：

```
python -m pip install flask
```

特定版本安装：

```
pip install flask==3.0.3
```

### 3. Python虚拟环境

##### 3.1 什么是Python虚拟环境

虚拟环境（Virtual Environment）是Python开发中非常重要的概念，尤其对于从Java转来的开发者，可以把它类比为**Java中的Maven/Gradle依赖隔离**或**Docker容器化思想**。

Python的虚拟环境包含：

* Python解释器副本
* 独立的第三方库安装目录
* 与其他项目完全隔离的依赖体系

类似于Java不同Maven项目有自己的pom.xml，不会互相干扰

***

##### 3.2 为什么需要虚拟环境

|   问题类型   |     Java解决方案      |      Python解决方案      |
| :----------: | :-------------------: | :----------------------: |
| 依赖版本冲突 |   各项目独立pom.xml   |  为每个项目创建虚拟环境  |
|   全局污染   | Maven本地仓库版本隔离 |     虚拟环境完全隔离     |
|  环境一致性  |    pom.xml锁定版本    | requirements.txt固定依赖 |

##### 3.3 如何使用Python虚拟环境

1. 创建虚拟环境（会在当前目录生成venv文件夹）**v**irtual+**env**ironment

   ```bash
   python -m venv venv
   ```

2. 激活虚拟环境

   ```bash
   # Linux
   source venv/bin/activate
   
   # windows
   venv\Scripts\activate
   ```

***

**!!!注意：**

`pip install `并不是全局生效的，会在对应的虚拟环境下才会生效

运行项目之前必须激活虚拟环境！

一般在.gitignore中排除venv/

***

3. 依赖的最佳实践

   **输出项目的依赖**：

   ```
   pip freeze>requirements.txt
   ```

   作用：输出当前项目所有依赖以及版本到requirements.txt文件中

​	**重新安装环境依赖or拿到别人项目后初始化：**

```
pip install -r requirements.txt
```





# 第一个Flask程序



```python
from flask import Flask

app= Flask(__name__)

@app.route('/')
def hello():
    return 'Hello Flask!'

if _name_=='_main_':
    app.run(debug=True)
```

### 详解

* `app=Flask(_name_)`

  **参数_name_:**表示当前模块名，用于确定应用根路径,这里name左右两侧都是**双下划线**

* `app.run(debug=True)`

  启动Flask开发服务器

### 进阶

1. 带参路由

   在route的参数里面末尾新增<type:value>,再在方法定义时传参即可

   ```python
   @app.route('/api/user/<int:id>')
   def getUser(id):
       return f"User ID :{id}"
   ```

2. 返回JSON

   flask里面提供了JSON化工具`jsonify`

   ```python
   @app.route('/api/data')
   def get_data():
       return jsonify({"name": "张三", "age": 25})  # 自动设置Content-Type
   ```

   

# 认识Flask原理

### 1. WSGI 与 Flask：底层运行机制简介

Flask 并不是直接运行在操作系统或 Web 浏览器之间的，而是基于一个叫做 **WSGI（Web Server Gateway Interface）** 的协议。



#### 什么是WSGI

WSGI 是 Python Web 应用程序与 Web 服务器之间的一种接口标准。简单来说，WSGI 让 Flask、Django 这样用 Python 编写的 Web 应用，可以和 [Nginx](https://zhida.zhihu.com/search?content_id=261000129&content_type=Article&match_order=1&q=Nginx&zhida_source=entity)、Apache、[Gunicorn](https://zhida.zhihu.com/search?content_id=261000129&content_type=Article&match_order=1&q=Gunicorn&zhida_source=entity) 等 Web 服务器协作。



路径：

> 浏览器发来请求 → Web 服务器（如 Gunicorn）接收 → 调用 Flask 应用（遵守 WSGI 协议） → 返回响应 → 浏览器接收结果。

### 2. Flask核心模块

##### 2.1 Flask应用对象--app

app是整个项目的**入口**和**大脑**

通过:

```python
app=Flask(__name__)
```

创建一个Flask应用对象，负责：

* 路由注册：如app.route()

* 配置管理

* 中间件处理

* 请求调度

  

##### 2.2 请求对象--request

request里面封装了浏览器发来的 HTTP 请求，包含所有请求相关的信息：

```python
`from flask import request

@app.route("/hello")
def hello():
    name = request.args.get("name", "Guest")
    return f"Hello, {name}
```

`request.method`, `request.headers`, `request.form`, `request.json` 等属性可以获取不同类型的数据。****



##### 2.3 响应对象--response

返回的内容最终都会被 Flask 封装成一个 `Response` 对象：

```python
@app.route("/raw")
def raw():
    return Response("This is a raw response", status=200, mimetype="text/plain")
```

一般我们直接返回字符串或者JSON对象的时候，Flask会自动封装



##### 2.4 蓝图--Buleprint

用于把大型项目的路由和逻辑拆分成多个模块（类比 Django 的 app），让你的应用更易维护。

```python
from flask import Blueprint

user_bp = Blueprint("user", __name__)

@user_bp.route("/login")
def login():
    return "User Login"
```

然后用 `app.register_blueprint(user_bp)` 注册到主应用



##### 2.5 上下文对--context

Flask 是线程安全的，它通过上下文对象（`request`, `g`, `session`, `current_app`）让你可以在任意函数中访问这些“全局”数据，而不需要层层传参。

```python
from flask import g

@app.before_request
def load_user():
    g.user = "current_user"
```

g相当于单次请求中的“全局变量”，能在单词请求中调用，但是和其他请求是互相隔离的



##### 从单文件到 Blueprint 的演化

初学者通常从一个单文件 Flask 项目开始：

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route("/")
def index():
    return "Hello Flask"
```

虽然这种方式上手简单，但随着业务增多，会导致：

- 路由混乱
- 配置难以统一
- 逻辑难以复用

所以我们推荐你逐渐过渡到模块化项目结构：

```python
myapp/
├── app/
│   ├── __init__.py          # 创建 Flask app，注册蓝图
│   ├── views/               # 路由逻辑放这里
│   │   ├── user.py          # 用户模块的蓝图
│   │   └── post.py          # 文章模块的蓝图
│   ├── models/              # 数据模型（可用于 ORM）
│   └── config.py            # 配置文件
├── run.py                   # 程序入口，运行服务
```

**详细内容为：**

**1）在路由实现里增加Blueprint，比如用户模块**

```text
# app/views/user.py
from flask import Blueprint

user_bp = Blueprint('user', __name__, url_prefix='/user')

@user_bp.route('/login')
def login():
    return "用户登录"

@user_bp.route('/register')
def register():
    return "用户注册"
```

你可以看到，我们用 `Blueprint()` 创建了一个Blueprint `user_bp`，并通过 `url_prefix='/user'` 统一设置前缀。

**2）注册Blueprint到主 app**

```python
# app/__init__.py
from flask import Flask

def create_app():
    app = Flask(__name__)

    # 蓝图导入
    from app.views.user import user_bp
    from app.views.post import post_bp  # 假设你有这个

    # 蓝图注册
    app.register_blueprint(user_bp)
    app.register_blueprint(post_bp)

    return app
```

**3）程序入口**

启动后就能看到不同的API了

```python
# run.py
from app import create_app

app = create_app()

if __name__ == "__main__":
    app.run(debug=True)
```

### 3 Flask 与 Web 项目架构的结合 —— 五层结构详解

在实际开发中，初学者常常把所有逻辑都堆在一个 `app.py` 文件里：路由、逻辑、数据库操作全部混在一起，短期看似简单，长期却极难维护。

本文里我们将结合上一篇文章介绍的 Python Web 项目的“五层结构”分层架构，帮助你从一开始就组织清晰的代码结构，模拟真实 Web 项目的开发方式，打好长期可维护的基础。

#### 3.1 路由层（Router）

##### 1）路由的作用

- 接收客户端请求的 URL 和方法（GET/POST）
- 将请求分发给相应的控制器（Controller）

##### 2）Flask 中的基本写法

```python
@app.route('/hello')
def hello():
    return "Hello World"
```

##### 3）GET/POST 请求注册

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        # 处理登录表单提交
        ...
    else:
        # 显示登录页面
        ...
```

##### 4）RESTful 风格初识

```python
@app.route('/user/<int:user_id>')
def get_user(user_id):
    return f"User ID is {user_id}"
```

- **路径参数**：通过 URL 中的部分识别资源
- **查询参数**：`/search?q=flask` 这种形式通过 `request.args.get('q')` 获取

#### 3.2 控制器层（Controller）

##### 1）什么是 Controller？

控制器接收来自路由的请求，调用业务逻辑层（Service），最后返回响应结果。

##### 2）为什么要解耦？

把所有逻辑都写在路由函数中会导致代码臃肿、难以维护，使用控制器层可以清晰分工。

##### 3）使用 Blueprint 管理多个模块

```python
# user_controller.py
from flask import Blueprint, request
from services.user_service import register_user

user_bp = Blueprint('user', __name__, url_prefix='/user')

@user_bp.route('/register', methods=['POST'])
def register():
    data = request.json
    return register_user(data)
# app/__init__.py
from flask import Flask
from controllers.user_controller import user_bp

def create_app():
    app = Flask(__name__)
    app.register_blueprint(user_bp)
    return app
```

#### 3.3 业务逻辑层（Service）

##### 1）为什么要有 Service 层？

- 保持控制器简洁
- 聚合业务逻辑
- 便于复用与测试

##### 2）示例：用户注册逻辑

```python
# services/user_service.py
from models.user import User
from repository.user_repository import UserRepository

def register_user(data):
    if UserRepository.get_user_by_email(data['email']):
        return {'msg': '邮箱已注册'}, 400
    user = User(**data)
    UserRepository.save(user)
    return {'msg': '注册成功'}, 201
```

##### 拆分建议

- 每个模块一个 service 文件，如 `user_service.py`、`blog_service.py`
- Service 层只处理“业务”，不关心 HTTP，也不关心数据库细节

#### 3.4 数据模型层（Model）

##### 1）使用 [Flask-SQLAlchemy](https://zhida.zhihu.com/search?content_id=261000129&content_type=Article&match_order=1&q=Flask-SQLAlchemy&zhida_source=entity) 定义模型类

```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password = db.Column(db.String(128), nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
```

##### 2）初始化数据库

```python
# app/__init__.py
db.init_app(app)
with app.app_context():
    db.create_all()
```

##### 3）封装模型方法

```python
class User(db.Model):
    ...
    @classmethod
    def create_user(cls, email, password):
        user = cls(email=email, password=password)
        db.session.add(user)
        db.session.commit()
        return user
```

#### 3.5 数据存储层（Storage / DAO）

##### 1）与模型层的区别？

- 模型层是数据库结构定义
- 存储层封装的是**数据库访问逻辑**

##### 2）好处

- 查询逻辑集中管理
- 替换存储方式时只需改 DAO 层（比如从 MySQL 换成 Redis）

##### 3）示例

```python
# repository/user_repository.py
from models.user import User
from models import db

class UserRepository:
    @staticmethod
    def get_user_by_email(email):
        return User.query.filter_by(email=email).first()

    @staticmethod
    def save(user):
        db.session.add(user)
        db.session.commit()
```

### 4 部署准备与常见问题简介

本节我们将从开发环境过渡到生产环境，简单介绍一个 Flask 项目如何准备上线部署，并回答初学者常见的几个问题，帮助你少走弯路。

#### 4.1 如何运行一个可部署的 Flask 项目（使用 Gunicorn + Nginx 简介）

Flask 自带的开发服务器 `app.run()` 仅适合本地调试，不推荐线上使用。上线部署推荐使用更稳定的 WSGI 容器，比如 **Gunicorn** 配合 **Nginx** 作为前端代理。

##### 1）示例部署结构

假设你的项目目录如下：

```python
myapp/
├── app/
│   └── ...
├── run.py
├── requirements.txt
```

##### 2）安装 Gunicorn

```python
pip install gunicorn
```

##### 3）使用 Gunicorn 启动 Flask 项目

```python
gunicorn -w 4 -b 0.0.0.0:8000 run:app
```

说明：

- `-w 4`：启动 4 个 worker 进程（具体的含义，后面深度教程里会讲解）
- `-b`：绑定到指定地址和端口
- `run:app`：表示从 `run.py` 文件中导入变量 `app`

##### 4）配合 Nginx 使用（简要说明）

- **Nginx** 用作反向代理，监听 80 端口
- 转发请求到 Gunicorn 的 8000 端口
- 可以加上静态资源路径管理、HTTPS 证书等功能

```python
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        include proxy_params;
        proxy_redirect off;
    }
}
```

#### 4.2 使用 .env 管理数据库密码、环境变量

在开发时，我们经常把数据库连接写死在配置文件里，比如：

```python
SQLALCHEMY_DATABASE_URI = "mysql+pymysql://root:123456@localhost/dbname"
```

这种方式上线时非常危险，容易泄漏密码。

**更好的方式：使用 `.env` 文件管理配置**

```bash
FLASK_ENV=production
DATABASE_URL=mysql+pymysql://user:password@localhost/dbname
SECRET_KEY=your-secret-key
```

在代码中读取：

```python
from dotenv import load_dotenv
load_dotenv()

import os
SQLALCHEMY_DATABASE_URI = os.getenv("DATABASE_URL")
SECRET_KEY = os.getenv("SECRET_KEY")
```

**安装依赖：**

```bash
pip install python-dotenv
```

#### 4.3 Flask 调试模式常见错误与解决方式

| 问题                                          | 说明与解决办法                                               |
| --------------------------------------------- | ------------------------------------------------------------ |
| 代码改了没生效                                | 没开 debug 模式？确保 app.run(debug=True)                    |
| 报错 “Working outside of application context” | 缺少 app.app_context() 包裹上下文                            |
| 数据库连接失败                                | 数据库没启动？配置有误？建议先 print(SQLALCHEMY_DATABASE_URI) 检查 |
| 前端页面 404                                  | 检查 @app.route() 的路径，是否用了 /、/index 等              |

### 4.4 推荐部署路径

##### **1）新手推荐：使用 Docker 快速部署**

编写一个简单的 `Dockerfile`：

```text
FROM python:3.10

WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:8000", "run:app"]
```

配合 Docker Compose 可管理数据库容器和应用容器。

##### **2）云服务推荐：**

- **国内**：阿里云、腾讯云、华为云
- **国外**：Render、Railway、Heroku（适合免费试用）

##### **3）本地服务器部署：**

- 推荐使用 Linux（如 Ubuntu）
- 使用 Supervisor 管理 Gunicorn 进程
- 配合 Nginx 做负载均衡和静态资源代理

