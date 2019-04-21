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

### template
 目录底下创建 templates 目录并建立 hello.html文件

 ```
 -- templates
    -- hello.html
 ```

 settings.py文件里指定了末班文件的路径。

 Django拥有强大的模板功能，使用该模板能设计出强大的Web页面。

 ``` html
 # /template/year_archive.html
{% block title %}Books for {{ year }}{% endblock %}

{{% block content %}}
<h1>Articles for {{years}}</h1>

{% for book in book_list %}
    <p>{{ book.name }}</p>
    <p>Published {{ book.pub_date|date:"F j, Y" }}</P>
{% endfor %}
{% end block %}

 ```

 year_archive.html 是 html 模板。 使用带基本逻辑声明的模板语言，如{% for book in book_list %}，它试图将函数返回的结果显示在网页上。
Django的这种结构我们称之为MTV模式：M代表模型(Model)，T代表模板(Template)，V代表视图(View)。这三个分别代表了三种不同功能的Web应用，各司其职，又彼此合作。 大家一定听说过MVC模式(Mode-View-Controller)，MTV模式本质上与MVC相同，都是让各组件保持低耦合的关系。
