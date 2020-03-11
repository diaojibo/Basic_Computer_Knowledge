## Pipenv管理项目
Pipenv是Kenneth Reitz在2017年1月发布的Python依赖管理工具，现在由PyPA维护。是现在Python官方推荐的包管理工具，集成virtualenv、pyenv 和 pip 三者的功能于一身。利用Pipenv，将可以使项目的python环境跟外部隔离开来，根据Pipfile自动安装管理依赖，不用污染外部的Python环境。

### 使用

安装pipenv

```
pip install pipenv
```

#### 安装指定版本的虚拟环境

```
pipenv install     // pipenv install --three
```

如果指定了 --two 或者 --three 选项参数，则会使用 python2 或者 python3 的版本安装，否则将使用默认的 python 版本来安装。当然也可以指定准确的版本信息：

```
$ pipenv install --python 3
$ pipenv install --python 3.6
$ pipenv install --python 2.7.14
```

这时候在当前 new_env 环境下生成 Pipfile 和 Pipfile.lock 两个环境初始化文件。

#### 进入环境，退出环境

```
// 进入
pipenv shell

// 退出
exit
```

#### 安装第三方包

```
pipenv install urllib3

// 如果想安装指定版本，则pipenv install urllib3==1.22
```

此时，Pipfile 里有最新安装的包文件的信息，如名称、版本等。用来在重新安装项目依赖或与他人共享项目时，你可以用 Pipfile 来跟踪项目依赖。

Pipfile 是用来替代原来的 requirements.txt 的，内容类似下面这样。source 部分用来设置仓库地址，packages 部分用来指定项目依赖的包，dev-packages 部分用来指定开发环境需要的包，这样分开便于管理。

```
$ cat Pipfile
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
"urllib3" = "*"

[dev-packages]

[requires]
python_version = "3.6"
```

Pipfile.lock 则包含你的系统信息，所有已安装包的依赖包及其版本信息，以及所有安装包及其依赖包的 Hash 校验信息。

```
$ Pipfile.lock
{
    "_meta": {
        "hash": {
            "sha256": "af58f3510cb613d4d9241128f9a0ceb9bb936ad907543e23ad8317011dcb6715"
        },
        "pipfile-spec": 6,
        "requires": {
            "python_version": "3.6"
        },
        "sources": [
            {
                "name": "pypi",
                "url": "https://pypi.org/simple",
                "verify_ssl": true
            }
        ]
    },
    "default": {                
         "urllib3": {
            "hashes": [
                "sha256:a68ac5e15e76e7e5dd2b8f94007233e01effe3e50e8daddf69acfd81cb686baf",
                "sha256:b5725a0bd4ba422ab0e66e89e030c806576753ea3ee08554382c14e685d117b5"
            ],
            "index": "pypi",
            "version": "==1.23"
        }
    },
    "develop": {}
}
```


卸载和更新包则有：

```
pipenv uninstall urllib3 //或者 pipenv uninstall --all

pipenv update urllib3 // pipenv update是更新所有

pipenv install --dev django // 只安装作开发用
```

查看依赖关系树：

```
pipenv graph
```

#### 在虚拟环境中运行命令

```
pipenv run python manage.py runserver
```

执行pipenv即可在虚拟环境里跑脚本啦

#### 冻结pipfile(锁定依赖版本)
冻结就相当于将项目所使用的第三方库列表进行打包输出，类似于virtualenv中生成requirements.txt文件。

通过更新Pipfile.lock来冻结库名称及其版本，以及其依赖关系的列表。需要使用lock参数

```
pipenv lock
```

如果另一个用户拷贝了你的项目，他们只需要安装Pipenv，然后：

```
pipenv install
```

Pipenv会在项目文件夹下自动寻找Pipfile和Pipfile.lock文件，创建一个新的虚拟环境并安装必要的软件包。

也就是说pipenv install的时候有三种逻辑：


 - 如果目录下没有Pipfile和Pipfile.lock文件，表示创建一个新的虚拟环境；
 - 如果有，表示使用已有的Pipfile和Pipfile.lock文件中的配置创建一个虚拟环境；
 - 如果后面带诸如django这一类库名，表示为当前虚拟环境安装第三方库。

#### 定位环境

```
pipenv --where // 定位项目路径:

pipenv --venv // 定位虚拟环境

pipenv --py // 定位Python解释器：
```