## Python日期和时间

### datetime类
python用这个datetime类来表示时间概念。

 - datetime 模块定义了下面的几个类：
 - datetime.date：表示日期的类。常用的属性有year, month, day；
 - datetime.time：表示时间的类。常用的属性有hour, minute, second, microsecond；
 - datetime.datetime：表示日期时间。
 - datetime.timedelta：表示时间间隔，即两个时间点之间的长度。
 - datetime.tzinfo：与时区有关的相关信息

所以我们`import datetime`之后，想知道年月日的信息，这个类是`datetime.date`类，如果想知道时分秒，那就是datetime.time类了。

我们一般通过以下调用得到一个datetime.datetime类的日期对象：

``` python
import datetime
print(datetime.datetime.today())  # 获取当前当地时间

import datetime
print(datetime.datetime.now())    # 获取当前当地时间，now可以有参数，默认为 NONE
```

一些获取时间的实例代码：

``` python
import datetime, time
t = time.time()                   # 获取时间戳
print(t)
print(datetime.datetime.today())  # 获取当前当地时间
print(datetime.datetime.now())    # 获取当前当地时间，now可以有参数，默认为 NONE
print(datetime.datetime.fromtimestamp(t))   # 通过时间戳，获得一个时间对象
print(datetime.datetime.strptime('2017-9-1 18:19:59', '%Y-%m-%d %H:%M:%S')) # 将字符串格式化为赌铲对象
print(datetime.datetime.date(datetime.datetime.now()))  # 返回时间对象的年月日
print(datetime.datetime.time(datetime.datetime.now()))  # 返回时间对象的时分秒
print(datetime.datetime.weekday(datetime.datetime.now()))  # 返回当天是本周的第几天，取值[0,6]
print(datetime.datetime.isoweekday(datetime.datetime.now())) # 返回当天是本周的第几天，取值[1,7]
print(datetime.datetime.ctime(datetime.datetime.now()))  # 将 时间对象 返回 时间字符串
print(datetime.datetime.now().strftime('%a, %b %d %H:%M'))  # 将时间对象返回 时间字符串
```

获取timestamp用以下代码：

```
dt = datetime(2017,7,19,8,30)   # 用指定日期时间创建datetime
dt.timestamp()                  # 把datetime转换为timestamp
#1500424200.0
```

### datetime加减

对日期和时间进行加减实际上就是把datetime往后或往前计算，得到新的datetime。加减可以直接用+和-运算符，不过需要导入datetime.timedelta这个类：

``` python
now = datetime.now()
#now
datetime.datetime(2017, 7, 19, 0, 53, 2, 335063)
now + timedelta(hours=10)
#datetime.datetime(2017, 7, 19, 5, 53, 2, 335063)
now - timedelta(days=2)
#datetime.datetime(2017, 7, 17, 0, 53, 2, 335063)
now + timedelta(days=3, hours=10)
#datetime.datetime(2017, 7, 22, 10, 53, 2, 335063)
```