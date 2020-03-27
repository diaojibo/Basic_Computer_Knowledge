## logrus日志切割

logrus是个不错的开源日志库，可以配合 **file-rotatelogs**库来进行日志切割。



logrus 本身不支持日志轮转切割功能，需要配合 file-rotatelogs 包来实现，防止日志打满磁盘。file-rotatelogs 实现了 io.Writer 接口，并且提供了文件的切割功能，其实例可以作为 logrus 的目标输出，两者能无缝集成，这也是 file-rotatelogs 的设计初衷


一个简单的示例代码就搞定一切：



```go
package log

import (
	"time"

	"log"

	rotatelogs "github.com/lestrrat-go/file-rotatelogs"
	"github.com/sirupsen/logrus"
)

var Log = logrus.New()

func init() {
	path := "../log/server.log"

	/* 日志轮转相关函数
	`WithLinkName` 为最新的日志建立软连接
	`WithRotationTime` 设置日志分割的时间，隔多久分割一次
	WithMaxAge 和 WithRotationCount二者只能设置一个
	  `WithMaxAge` 设置文件清理前的最长保存时间
	  `WithRotationCount` 设置文件清理前最多保存的个数
	*/

	// 保存60天, 24小时滚动一次
	writer, err := rotatelogs.New(
		path+".%Y%m%d",
		rotatelogs.WithLinkName(path),
		rotatelogs.WithMaxAge(time.Duration(24*60)*time.Hour),
		rotatelogs.WithRotationTime(time.Duration(24)*time.Hour),
	)

	if err != nil {
		log.Fatal("Some error occur %v", err)
	}

	// 设置一下log的输出，哇，就打工告成了
	Log.SetOutput(writer)
	Log.SetLevel(logrus.DebugLevel)

	// 设置一下log的一些参数
	myFormatter := new(logrus.TextFormatter)
	myFormatter.FullTimestamp = true
	myFormatter.TimestampFormat = "2006-01-02 15:04:05.000"
	myFormatter.DisableColors = false
	Log.SetFormatter(myFormatter)

}

```



上面代码就相当于把log单例对象导出了，顺便还设置了一些输出时间的格式等等参数。