## gin

Gin是用 Go 开发的一个微框架，类似 Martinier 的API，重点是小巧、易用、性能好很多，也因为 [httprouter](https://github.com/julienschmidt/httprouter) 的性能提高了40倍。

我们go mod init新建好自己的工程以后，就可以引入gin了

```shell
go get -u github.com/gin-gonic/gin
```

记住在go的项目内，我们用go get -u就能更新现有的依赖。

