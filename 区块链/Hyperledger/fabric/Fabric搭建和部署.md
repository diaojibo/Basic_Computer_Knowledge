# Fabric搭建和部署
fabric就不多做介绍了，直奔主题，如何部署Fabric网络。

## 安装go
首先就是安装Go语言环境。

去到golang的官网：`https://golang.org/dl/`

![](image/fabric0.png)

得到下载网址，然后wget到linux下：

```
wget https://dl.google.com/go/go1.11.2.linux-amd64.tar.gz

tar -xvf go1.11.2.linux-amd64.tar.gz

// 移动到标准软件安装目录下
sudo mv go /usr/local 

export GOPATH=/usr/local/go
export 
```

## 安装docker
接下来我们安装docker

我们可以根据官方文档来进行安装：[文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/#prerequisites)

由于命令时常变化，就不贴安装方法了。注意安装docker是要先建立仓库的。

输出docker --version查看版本就知道是否成功了。