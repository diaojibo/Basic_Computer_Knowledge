## apache配置基础
Apache 是一个可靠、流行且易于配置的 Web 服务器。

Apache 主要的配置文件是httpd.conf

### 基本配置


 - ServerRoot "/mnt/software/apache2" #你的apache软件安装的位置。其它指定的目录如果没有指定绝对路径，则目录是相对于该目录。
 - PidFile logs/httpd.pid #第一个httpd进程(所有其他进程的父进程)的进程号文件位置。
 - Listen 80 #服务器监听的端口号。
 - ServerName www.clusting.com:80 #主站点名称（网站的主机名）。
 - ServerAdmin admin@clusting.com #管理员的邮件地址。
 - DocumentRoot "/mnt/web/clusting" #主站点的网页存储位置。

针对主站点，会有一个xml项来进行访问控制：

```
<Directory "/mnt/web/clusting"> 
    Options FollowSymLinks 
    AllowOverride None 
    Order allow,deny 
    Allow from all 
</Directory> 
```

Options：配置在特定目录使用哪些特性，常用的值和 基本含义如下：

 - ExecCGI: 在该目录下允许执行CGI脚本。

 - FollowSymLinks: 在该目录下允许文件系统使用符号连接。

 - Indexes: 当用户访问该目录时，如果用户找不到DirectoryIndex指定的主页文件(例如index.html),则返回该目录下的文件列表给用户。
 - SymLinksIfOwnerMatch: 当使用符号连接时，只有当符号连接的文件拥有者与实际文件的拥有者相同时才可以访问。

AllowOverride：允许存在于.htaccess文件中的指令类型(.htaccess文件名是可以改变的，其文件名由AccessFileName指令决定)

Order：控制在访问时Allow和Deny两个访问规则哪个优先

DirectoryIndex index.html index.htm index.php #主页文件的设置