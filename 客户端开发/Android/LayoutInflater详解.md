## LayoutInflater详解
我们写的 xml 布局配置文件都是通过 LayoutInflater 来 inflate 成具体的 View 对象的。它的作用类似于findViewById()。不同点是LayoutInflater是用来找res/layout/下的xml布局文件，并且实例化；而findViewById()是找xml布局文件下的具体widget控件（如Button，TextView等等）。

### 使用场景
 - 对于一个没有被载入或者想要动态载入的界面，都需要使用LayoutInflater.inflater()来载入。
 - 对于一个已经载入的界面，可以使用Activity.findViewById()方法来获取其中的界面元素。

### 获取LayoutInflater实例的三种方式

``` java
// 方式1
// 调用Activity的getLayoutInflater()
LayoutInflater inflater = getLayoutInflater();
// 方式2
LayoutInflater inflater = LayoutInflater.from(context);
// 方式3
LayoutInflater inflater = (LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```

其实，这三种方式本质上是一样的，从源码中可以看出
