## 文件管理
记录一下Django 1.X版本下的文件管理，之后可能会过时，那就重新去查文档。

默认情况下，Django 本地存储（上传的）文件，通过 **MEDIA_ROOT 和 MEDIA_URL 设置来实现**。

``` python
# settings.py

MEDIA_ROOT = os.path.abspath('../media')
MEDIA_URL = '/media/'

# urls.py

from django.conf.urls.static import static

urlpatterns = [
    # ...
] + static(
    settings.MEDIA_URL,
    document_root=settings.MEDIA_ROOT,
)
```

### File对象
Django 内部总是用 File 对象来封装并表示文件，这是在 Python 原生 file 对象上的一个轻量封装。

绝大多数时候，你都只会使用 django 给你的 File 对象（即例如上面的获取一个模型对象的文件字段。）

如果真要自己搞一个，最简单的办法就是通过 Python 原生 File 对象自己创建一个：

``` python
>>> from django.core.files import File

# Create a Python file object using open()
>>> f = open('/tmp/hello.world', 'w')
>>> myfile = File(f)
```

### storage文件存储
Django 的默认文件存储是通过 DEFAULT_FILE_STORAGE 的 settings 配置决定的，如果不额外显式指定存储系统（在调用的时候），我们默认就会用这个。

我们可以通过文件存储类来存储文件

``` python
>>> from django.core.files.storage import default_storage
>>> from django.core.files.base import ContentFile

>>> path = default_storage.save('/path/to/file', ContentFile('new content'))
>>> path
'/path/to/file'

>>> default_storage.size(path)
11
>>> default_storage.open(path).read()
'new content'

>>> default_storage.delete(path)
>>> default_storage.exists(path)
False
```