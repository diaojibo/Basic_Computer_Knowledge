## dockerfile编写
Docker可以通过读取Dockerfile指令自动生成镜像 。Dockerfile文件是一个文本文档，其中包含用户可以在命令行上调用以构建镜像的所有命令。通过使用**docker build**， 用户可以创建一个**自动构建，它会连续执行Dockerfile文件定义的命令行指令**。

### 用法
**docker build** 命令将会根据一个Dockerfile文件和上下文构来建镜像。构建的上下文是在指定位置的文件集合，可以是PATH或URL。PATH是本地文件系统上的一个目录。URL是一个Git仓库的位置

上下文会被递归地处理。所以，PATH 可以包括任何子目录，且URL可以包含存储库及其子模块。这个例子展示了一个使用当前目录作为上下文的构建命令：

```
$ docker build .
Sending build context to Docker daemon  6.51 MB
...
```

总而言之，在镜像构建的时候，可以传递一个上下文路径，这个路径下的文件都可以被传递进镜像来处理的。也就是能被docker环境感知。

可以使用 -f 参数指定 docker build指向文件系统中任何位置的Dockerfile 文件。

```
docker build -f /path/to/a/Dockerfile .
```

Docker守护进程逐个运行Dockerfile中的指令，在必要时将每条指令的结果提交给新镜像，最后输出新镜像的ID 。

**zwlj：切记，docker build命令配合docker file是用来创建镜像的。镜像的概念我们在基础笔记里有提到，它是一个模板，而不是一个运行的实例进程。我们需要根据镜像模板，从中才能跑出一个容器实例**

#### dockerignore文件优化上下文
构建镜像时，Docker 需要先准备context ，将所有需要的文件收集到进程中。默认的context包含 Dockerfile 目录中的所有文件，但是实际上，我们并不需要.git 目录，node_modules 目录等内容。 .dockerignore 的作用和语法类似于 .gitignore，可以忽略一些不需要的文件，这样可以有效加快镜像构建时间，同时减少 Docker 镜像的大小。

```
# .dockerignore文件
.git/
node_modules/
```

### dockerfile指令
接下来看看dockerfile编写时的指令语法


#### FROM
Docker 按顺序运行Dockerfile 中的指令。**一个Dockerfile 文件必须从FROM指令开始。该FROM指令指定了您正在构建的基础镜像**。FROM只能在该行所有参数之前。

所以请先记住**FROM**

#### RUN
在shell或者exec的环境下执行的命令。RUN指令会在新创建的镜像上添加新的层面

```
RUN 《command》
```

#### CMD
CMD：提供了容器默认的执行命令。 Dockerfile只允许使用一次CMD指令。 使用多个CMD会抵消之前所有的指令，只有最后一个指令生效。

CMD有三种形式：

```
CMD ["executable","param1","param2"]
CMD ["param1","param2"]
CMD command param1 param2
```

#### EXPOSE
指定容器在运行时监听的端口。语法如下：

```
EXPOSE <port>;
```

### ADD
从当前目录复制文件到容器. 会自动处理目录, 压缩包等情况.

#### COPY
从当前目录复制文件到容器. 只是单纯地复制文件.

#### WORKDIR
指定RUN、CMD与ENTRYPOINT命令的工作目录(注意这个目录是容器里的目录)。语法如下：

```
WORKDIR /path/to/workdir
```

#### VOLUME
VOLUME：授权访问从容器内到主机上的目录。语法如下：

```
VOLUME ["/data"]
```