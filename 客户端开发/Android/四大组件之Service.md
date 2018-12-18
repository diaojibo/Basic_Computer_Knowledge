## Service详解
Service作为Android四大组件之一，在每一个应用程序中都扮演着非常重要的角色。它主要用于在**后台处理一些耗时的逻辑**，或者去执行某些需要长期运行的任务。必要的时候我们甚至可以在程序退出的情况下，**让Service在后台继续保持运行状态**。


### 使用Service

启动Service的方法和启动Activity很类似，都需要借助Intent来实现，下面我们就通过一个具体的例子来看一下。

创建一个服务非常简单，只要**继承Service**，并实现onBind()方法(只有这个是必须的)

新建一个MyService继承自Service，并重写父类的`onCreate()`、`onStartCommand()`和`onDestroy()`方法，如下所示：

``` java
public class MyService extends Service {

    public static final String TAG = "MyService";

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate() executed");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand() executed");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy() executed");
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

}
```

然后打开或新建MainActivity作为程序的主Activity，在里面加入启动Service和停止Service的逻辑

``` java
@Override
public void onClick(View v) {
    switch (v.getId()) {
    case R.id.start_service:
        Intent startIntent = new Intent(this, MyService.class);
        startService(startIntent);
        break;
    case R.id.stop_service:
        Intent stopIntent = new Intent(this, MyService.class);
        stopService(stopIntent);
        break;
    default:
        break;
    }
}
```

在Start Service按钮的点击事件里，我们构建出了一个Intent对象，并调用startService()方法来启动MyService。然后在Stop Serivce按钮的点击事件里，我们同样构建出了一个Intent对象，并调用stopService()方法来停止MyService。代码的逻辑非常简单，相信不需要再多做解释。


另外需要注意，项目中的每一个Service都必须在AndroidManifest.xml中注册才行，所以还需要编辑AndroidManifest.xml文件

``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.servicetest"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="14"
        android:targetSdkVersion="17" />

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >

    ……

        <service android:name="com.example.servicetest.MyService" >
        </service>
    </application>

</manifest>
```

普通的Service的**生命周期**很简单，分别为onCreate、onStartCommand、onDestroy这三个。当我们startService()的时候，首次创建Service会回调onCreate()方法，然后回调onStartCommand()方法，再次startService()的时候，就只会执行onStartCommand()。服务一旦开启后，我们就需要通过stopService()方法或者stopSelf()方法关闭服务，这时就会回调onDestroy()。


### Service和Activity进行通信

单纯按照上面方法启动service，无法在Activity中进行管理，相当于直接开启了一个服务以后就互不相干的。

可以通过重写onBind方法来实现和Activity的绑定。

#### 简单案例

我们重写一下上面的Service代码。重写onBind方法和新添加进一个Binder类子类的对象


``` java
public class MyService extends Service {

    public static final String TAG = "MyService";

    private MyBinder mBinder = new MyBinder();

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate() executed");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand() executed");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy() executed");
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    class MyBinder extends Binder {

        public void startDownload() {
            Log.d("TAG", "startDownload() executed");
            // 执行具体的下载任务
        }

    }

}
```

上面代码，我们定义了一个内部类，实现了Binder类，这个MyBinder类里定义了一个startDownload公有方法。

在onBind方法中，我们返回了一个myBinder类的成员实例。

接着我们在MainActivity里，添加以下的代码。

``` java
private MyService.MyBinder myBinder;

private ServiceConnection connection = new ServiceConnection() {

    @Override
    public void onServiceDisconnected(ComponentName name) {
    }

    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        myBinder = (MyService.MyBinder) service;
        myBinder.startDownload();
    }
};
```

相关按钮的逻辑

``` java
public void onClick(View v) {
    switch (v.getId()) {
    case R.id.bind_service:
        Intent bindIntent = new Intent(this, MyService.class);
        bindService(bindIntent, connection, BIND_AUTO_CREATE);
        break;
    case R.id.unbind_service:
        unbindService(connection);
        break;
    default:
        break;
    }
}
```

Activity类里也添加了MyBinder成员对象，还有一个ServiceConnection对象，实例时重写了两个onServiceConnected方法，用来指导绑定服务时做什么操作。

然后按钮逻辑里，我们先用intent启动Service，然后用bindService进行绑定，一当绑定完成立刻就会执行connection 中的onServiceConnected方法，调用Mybinder中的startDownload方法。
