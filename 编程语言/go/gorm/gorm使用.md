## gorm使用

gorm构建在标准库database/sql上，所以需要先对database/sql有所了解。

**sql.DB**不是一个连接，**它是数据库的抽象接口**。它可以根据driver打开关闭数据库连接，管理连接池。

标准库文档的解释如下:

DB是一个数据库（操作）句柄，代表一个具有零到多个底层连接的连接池。它可以安全的被多个go程同时使用。
sql包会自动创建和释放连接；它也会维护一个闲置连接的连接池。如果数据库具有单连接状态的概念，该状态只有在事务中被观察时才可信。一旦调用了BD.Begin，返回的Tx会绑定到单个连接。当调用事务Tx的Commit或Rollback后，该事务使用的连接会归还到DB的闲置连接池中。连接池的大小可以用SetMaxIdleConns方法控制。

**zwlj：总而言之，标准库的sql.DB就是一个连接池，不是一个单连接啦，所以可以大胆的作为包全局变量，init成功以后大胆的去用**

### gorm操作

```go
//定义User模型，绑定users表，ORM库操作数据库，需要定义一个struct类型和MYSQL表进行绑定或者叫映射，struct字段和MYSQL表字段一一对应
//在这里User类型可以代表mysql users表
type User struct {
	//通过在字段后面的标签说明，定义golang字段和表字段的关系
	//例如 `gorm:"column:username"` 标签说明含义是: Mysql表的列名（字段名)为username
	//这里golang定义的Username变量和MYSQL表字段username一样，他们的名字可以不一样。
	Username string `gorm:"column:username"`
	Password string `gorm:"column:password"`
	//创建时间，时间戳
	CreateTime int64 `gorm:"column:createtime"`
}
```

如上其实就是对应了一张User表

```sql
CREATE TABLE `users` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增ID',
  `username` varchar(30) NOT NULL COMMENT '账号',
  `password` varchar(100) NOT NULL COMMENT '密码',
  `createtime` int(10) NOT NULL DEFAULT 0 COMMENT '创建时间',
   PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8
```



创建好连接以后，就可以开始orm的对应操作了：

```go
func main()  {
	//配置MySQL连接参数
	username := "root"  //账号
	password := "123456" //密码
	host := "127.0.0.1" //数据库地址，可以是Ip或者域名
	port := 3306 //数据库端口
	Dbname := "tizi365" //数据库名

	//通过前面的数据库参数，拼接MYSQL DSN， 其实就是数据库连接串（数据源名称）
	//MYSQL dsn格式： {username}:{password}@tcp({host}:{port})/{Dbname}?charset=utf8&parseTime=True&loc=Local
	//类似{username}使用花括号包着的名字都是需要替换的参数
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=utf8&parseTime=True&loc=Local", username, password, host, port, Dbname)
	//连接MYSQL
	db, err := gorm.Open("mysql", dsn)
	if err != nil {
		panic("连接数据库失败, error=" + err.Error())
	}

	//定义一个用户，并初始化数据
	u := User{
		Username:"tizi365",
		Password:"123456",
		CreateTime:time.Now().Unix(),
	}

	//插入一条用户数据
	//下面代码会自动生成SQL语句：INSERT INTO `users` (`username`,`password`,`createtime`) VALUES ('tizi365','123456','1540824823')
	if err := db.Create(u).Error; err != nil {
		fmt.Println("插入失败", err)
		return
	}

	//查询并返回第一条数据
	//定义需要保存数据的struct变量
	u = User{}
	//自动生成sql： SELECT * FROM `users`  WHERE (username = 'tizi365') LIMIT 1
	isNotFound := db.Where("username = ?", "tizi365").First(&u).RecordNotFound()
	if isNotFound {
		fmt.Println("找不到记录")
		return
	}
	//打印查询到的数据
	fmt.Println(u.Username,u.Password)

	//更新
	//自动生成Sql: UPDATE `users` SET `password` = '654321'  WHERE (username = 'tizi365')
	db.Model(User{}).Where("username = ?", "tizi365").Update("password", "654321")

	//删除
	//自动生成Sql： DELETE FROM `users`  WHERE (username = 'tizi365')
	db.Where("username = ?", "tizi365").Delete(User{})
}
```

