## hiredis使用



先来看看大致的接口



```cpp
/**连接数据库*/
redisContext *redisConnect(const char *ip, int port);
/**发送命令请求*/
void *redisCommand(redisContext *c, const char *format, ...);
void *redisCommandArgv(redisContext *c, int argc, const char **argv, const size_t *argvlen);
void redisAppendCommand(redisContext *c, const char *format, ...);
void redisAppendCommandArgv(redisContext *c, int argc, const char **argv, const size_t *argvlen);
/*释放资源*/
void freeReplyObject(void *reply);
void redisFree(redisContext *c);
```



### redisCommand

这个函数是一个带有不定参数的。可以按着format格式给出对应的参数，这就和printf函数类似。

返回值是一个void类型的指针，实际为指向一个redisReply类型的指针。

```cpp
/* This is the reply object returned by redisCommand() */
typedef struct redisReply {
    /*命令执行结果的返回类型*/
    int type; /* REDIS_REPLY_* */
    /*存储执行结果返回为整数*/
    long long integer; /* The integer when type is REDIS_REPLY_INTEGER */
    /*字符串值的长度*/
    size_t len; /* Length of string */
    /*存储命令执行结果返回是字符串*/
    char *str; /* Used for both REDIS_REPLY_ERROR and REDIS_REPLY_STRING */
    /*返回结果是数组的大小*/
    size_t elements; /* number of elements, for REDIS_REPLY_ARRAY */
    /*存储执行结果返回是数组*/
    struct redisReply **element; /* elements vector for REDIS_REPLY_ARRAY */
} redisReply;
```



我们需要根据type来判定返回结果的类型：

- REDIS_REPLY_STRING == 1:**返回值是字符串**,字符串储存在redis->str当中,字符串长度为redis->len。
- REDIS_REPLY_ARRAY == 2：**返回值是数组**，数组大小存在redis->elements里面，数组值存储在redis->element[i]里面。数组里面存储的是指向redisReply的指针，数组里面的返回值可以通过redis->element[i]->str来访问，**数组的结果里全是type==REDIS_REPLY_STRING的redisReply对象指针。**
- REDIS_REPLY_INTEGER == 3：**返回值为整数 long long。**
- REDIS_REPLY_NIL==4：**返回值为空表示执行结果为空。**
- REDIS_REPLY_STATUS ==5：**返回命令执行的状态**，比如set foo bar 返回的状态为OK，存储在str当中 reply->str == "OK"。
- REDIS_REPLY_ERROR ==6 ：**命令执行错误**,错误信息存放在 reply->str当中。

