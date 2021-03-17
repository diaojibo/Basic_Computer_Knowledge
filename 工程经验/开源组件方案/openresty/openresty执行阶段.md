## openresty执行阶段

一个请求，肯定是有生命周期，执行阶段的概念的。

openresty的执行阶段其实相当云nginx执行阶段的一个简化版

OpenResty 处理一个请求，它的处理流程请参考下图（从 Request start 开始）：

![openresty_phases](https://moonbingbing.gitbooks.io/openresty-best-practices/content/images/openresty_phases.png)

Nginx的请求处理阶段有11，其中最重要的也是最常见的3个阶段依次为`rewrite`, `access`,`content`.

- `init_by_lua` `init_by_lua_block`: 运行在Nginx loading-config 阶段，注册Nginx Lua全局变量，和一些预加载模块。是Nginx master进程在加载Nginx配置时执行。
- `init_worker_by_lua`: 在Nginx starting-worker阶段，即每个nginx worker启动时会调用，通常用来hook worker进程，并创建worker进行的计时器，用来健康检查，或者设置熔断记时窗口等等。
- `access_by_lua`: 在`access tail`阶段，用来对每次请求做访问控制，权限校验等等，能拿到很多相关变量。例如：请求体中的值，header中的值，可以将值添加到`ngx.ctx`, 在其他模块进行相应的控制
- `balancer_by_lua`: 通过Lua设置不同的负载均衡策略, 具体可以参考[lua-resty-balancer](https://github.com/openresty/lua-resty-balancer)
- `content_by_lua`: 在content阶段，即`content handler`的角色，即对于每个api请求进行处理，注意不能与proxy_pass放在同一个location下
- `proxy_pass`: 真正发送请求的一部分, 通常介于`access_by_lua`和`log_by_lua`之间
- `header_filter_by_lua`:在`output-header-filter`阶段，通常用来重新响应头部，设置cookie等，也可以用来作熔断触发标记
- `body_filter_by_lua`:对于响应体的content进行过滤处理
- `log_by_lua`:记录日志即，记录一下整个请求的耗时，状态码等



**不同的执行阶段应该要做不同的事情，当然是可以把所有事情都放到content_by_lua里面来做，但是最好是分开，比如打日志，或者打日志到远程，那就在log_by_lua流程里去做**