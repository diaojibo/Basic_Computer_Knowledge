## docker命令
要本地玩转docker，还是要把docker命令给理清楚。

我们一般会通过下载或者自己写dockerfile来获取镜像，也就是模板，然后在用docker run的命令产生容器实例。

### 先写非常无敌重要的docker run
docker run ：创建一个新的容器(也就是实例)并运行一个命令

docker run一些参数要知道

 - -d: 后台运行容器，并返回容器ID
 - -i: 以交互模式运行容器，通常与 -t 同时使用
 - -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
 - -p: 指定端口映射，格式为：`主机(宿主)端口:容器端口`
 - -p: 指定端口映射，格式为：主机(宿主)端口:容器端口
 - --volume , -v:	绑定一个卷

实例：使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。

```
docker run -p 80:80 -v /data:/data -d nginx:latest
```


### docker search
默认在远端的仓库docker hub(默认是官方源docker.io)搜索镜像(也就是模板)

```
$ docker search ubuntu
NAME DESCRIPTION STARS OFFICIAL AUTOMATED
ubuntu Ubuntu is a Debian-based Linux operating sys… 7861 [OK]
dorowu/ubuntu-desktop-lxde-vnc Ubuntu with openssh-server and NoVNC 190 [OK]
rastasheep/ubuntu-sshd Dockerized SSH service, built on top of offi… 156 [OK]
ansible/ubuntu14.04-ansible Ubuntu 14.04 LTS with ansible 93 [OK]
ubuntu-upstart Upstart is an event-based replacement for th… 87 [OK]
neurodebian NeuroDebian provides neuroscience research s… 50 [OK]
…
```

### docker images查看已有镜像

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              113a43faa138        2 weeks ago         81.2MB
```

### docker pull下载镜像

```
docker pull ubuntu:16.04
```

还可以指定镜像的版本。

### docker login
我们pull下载镜像的时候，是无需远端docker hub的权限的，但是如果我们想把镜像推到docker hub上，那就需要login了。

```
docker login --username=yourhubusername --email=youremail@company.com
```

### docker ps
列出正在运行的docker容器(实例)

```
$ docker ps
CONTAINER ID         IMAGE         COMMAND         CREATED           STATUS          PORTS             NAMES
6162daed25da         nginx  "nginx -g 'daemon of…" 7 minutes ago   Up 7 minutes  0.0.0.0:8080->80/tcp  hardcore_torvalds
```

### docker stop
停止容器，后面可以接容器的Name或者容器的实例id

```
$ docker stop my_nginx
my_nginx
```

### docker rmi
删除容器，也就是删除实例拉

```
docker rm [ 容器名 or 容器 id ]
```

### docker save
这个命令用于导入导出容器

将镜像 runoob/ubuntu:v3 生成 my_ubuntu_v3.tar 文档

```
runoob@runoob:~$ docker save -o my_ubuntu_v3.tar runoob/ubuntu:v3
runoob@runoob:~$ ll my_ubuntu_v3.tar
-rw------- 1 runoob runoob 142102016 Jul 11 01:37 my_ubuntu_v3.tar
```