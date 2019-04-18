## django基础
安装django `pip install django`,然后可以直接在命令行执行django-admin创建一个样例python工程。`django-admin startproject HelloWorld`

django项目标准结构：

```
$ tree
.
|-- HelloWorld
|   |-- __init__.py
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
`-- manage.py
```


 - HelloWorld: 项目的容器。
 - manage.py: 一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互。
 - HelloWorld/__init__.py: 一个空文件，告诉 Python 该目录是一个 Python 包。
 - HelloWorld/settings.py: 该 Django 项目的设置/配置。
 - HelloWorld/urls.py: 该 Django 项目的 URL 声明; 一份由 Django 驱动的网站"目录"。
 - HelloWorld/wsgi.py: 一个 WSGI 兼容的 Web 服务器的入口，以便运行你的项目。

接下来我们进入 HelloWorld 目录输入以下命令，启动服务器：

```
python3 manage.py runserver 0.0.0.0:8000
```