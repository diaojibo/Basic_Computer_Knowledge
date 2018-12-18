## AsyncTask处理异步消息
在Android中我们可以通过Thread+Handler实现多线程通信，一种经典的使用场景是：在新线程中进行耗时操作，当任务完成后通过Handler向主线程发送Message，这样主线程的Handler在收到该Message之后就可以进行更新UI的操作。上述场景中需要分别在Thread和Handler中编写代码逻辑。

为了**使得代码更加统一**，我们可以使用AsyncTask类。

AsyncTask是Android提供的一个助手类，**它对Thread和Handler进行了封装**，方便我们使用。Android之所以提供AsyncTask这个类，就是为了方便我们在后台线程中执行操作，然后将结果发送给主线程，从而在主线程中进行UI更新等操作。在使用AsyncTask时，我们**无需关注Thread和Handler**，AsyncTask内部会对其进行管理，这样我们就只需要关注于我们的业务逻辑即可。

### 详解

AsyncTask的内部封装了两个线程池

 - SerialExecutor
 - THREAD_POOL_EXECUTOR

 还封装了一个Handler(InternalHandler)。

其中SerialExecutor线程池用于任务的排队，让需要执行的多个耗时任务，按顺序排列，THREAD_POOL_EXECUTOR线程池才真正地执行任务，InternalHandler用于从工作线程切换到主线程。

AsyncTask的类声明如下：

``` java
public abstract class AsyncTask<Params, Progress, Result>
{}

```

AsyncTask是一个抽象泛型类。

其中，三个泛型类型参数的含义如下：

 - Params：开始异步任务执行时传入的参数类型；

 - Progress：异步任务执行过程中，返回下载进度值的类型；

 - Result：异步任务执行完成后，返回的结果类型；

**如果AsyncTask确定不需要传递具体参数，那么这三个泛型参数可以用Void来代替**

有了这三个参数类型之后，也就控制了这个AsyncTask子类各个阶段的返回类型，如果有不同业务，我们就需要再另写一个AsyncTask的子类进行处理。

#### AsyncTask核心方法

 - onPreExecute()

这个方法会在后台任务开始执行之间调用，**在主线程执行**。用于进行一些界面上的**初始化操作**，比如显示一个进度条对话框等。

 - doInBackground(Params...)

这个方法中的**所有代码都会在子线程中运行**，我们应该在这里去**处理所有的耗时任务**。

 - onProgressUpdate(Progress...)

当在后台任务中调用了publishProgress(Progress...)方法后，这个方法就很快会被调用，方法中携带的参数就是在后台任务中传递过来的。在这个方法中**可以对UI进行操作**，在主线程中进行，利用参数中的数值就可以对界面元素进行相应的更新。

 - onPostExecute(Result)

当doInBackground(Params...)执行完毕并通过return语句进行返回时，这个方法就很快会被调用。**返回的数据会作为参数传递到此方法中**，可以利用返回的数据来进行一些UI操作，**在主线程中进行**，比如说提醒任务执行的结果，以及关闭掉进度条对话框等。

上面几个方法的调用顺序：
onPreExecute() --> doInBackground() --> publishProgress() --> onProgressUpdate() --> onPostExecute()

#### 使用例子

这里我们模拟了一个下载任务，在doInBackground()方法中去执行具体的下载逻辑，在onProgressUpdate()方法中显示当前的下载进度，在onPostExecute()方法中来提示任务的执行结果。

耐心观察以下代码

``` java
class DownloadTask extends AsyncTask<Void, Integer, Boolean> {

    @Override
    protected void onPreExecute() {
        progressDialog.show();
    }

    @Override
    protected Boolean doInBackground(Void... params) {
        try {
            while (true) {
                int downloadPercent = doDownload();
                publishProgress(downloadPercent);
                if (downloadPercent >= 100) {
                    break;
                }
            }
        } catch (Exception e) {
            return false;
        }
        return true;
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        progressDialog.setMessage("当前下载进度：" + values[0] + "%");
    }

    @Override
    protected void onPostExecute(Boolean result) {
        progressDialog.dismiss();
        if (result) {
            Toast.makeText(context, "下载成功", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(context, "下载失败", Toast.LENGTH_SHORT).show();
        }
    }
}

```

这里实现了一个AsyncTask类，并且把方法把对应方法都实现了。

如果想要启动这个任务，只需要简单地调用以下代码即可：

``` java
new DownloadTask().execute();
```

#### 使用AsyncTask的注意事项
①异步任务的实例**必须在UI线程**中创建，即AsyncTask对象必须在UI线程中创建。

②execute(Params... params)方法**必须在UI线程中调用**。

③不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Progress... values)，onPostExecute(Result result)这几个方法。

④**不能在doInBackground(Params... params)中更改UI组件的信息**。

⑤一个任务实例**只能执行一次**，如果执行第二次将会抛出异常。
