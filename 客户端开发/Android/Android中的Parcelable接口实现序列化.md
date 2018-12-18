## Parcelable接口
android提供了一种新的类型：parcel（英文解释：包裹，小包），用来封装数据的容器，封装后的数据可以通过Intent或IPC传递，除了基本类型外，只有实现了Parcelable接口的类才能放入parcel中。


实现Parcelable就是为了进行序列化，那么，为什么要序列化？

 - **永久性保存对象**，保存对象的字节序列到本地文件中；
 - 通过序列化对象**在网络中**传递对象；
 - 通过序列化**在进程间**传递对象。

Android中实现序列化有两个选择：一是实现**Serializable接口**（是JavaSE本身就支持的），一是实现**Parcelable接口**（是Android特有功能，**效率比实现Serializable接口高效**，可用于Intent数据传递，也可以用于进程间通信（IPC））。

实现Serializable接口非常简单，声明一下就可以了，而实现Parcelable接口稍微复杂一些，但效率更高，推荐用这种方法提高性能。

可以大致了解下Parcelable接口的定义

``` java
public interface Parcelable
{
    //内容描述接口，基本不用管
    public int describeContents();
    //写入接口函数，打包
    public void writeToParcel(Parcel dest, int flags);
    //读取接口，目的是要从Parcel中构造一个实现了Parcelable的类的实例处理。因为实现类在这里还是不可知的，所以需要用到模板的方式，继承类名通过模板参数传入
    //为了能够实现模板参数的传入，这里定义Creator嵌入接口,内含两个接口函数分别返回单个和多个继承类实例
    public interface Creator<T>
    {
           public T createFromParcel(Parcel source);
           public T[] newArray(int size);
    }
}
```

### 实现Parcelable
要让对象实现这个接口，要重写以下几个方法。

1. 重写writeToParcel方法，将你的对象序列化为一个Parcel对象，即：将类的数据写入外部提供的Parcel中，打包需要传递的数据到Parcel容器保存，以便从 Parcel容器获取数据

2. 重写describeContents方法，内容接口描述，**默认返回0就可以**

3. **实例化静态内部对象CREATOR实现接口Parcelable.Creator**


以上，重写writeToParcel方法是为了完成序列化，完成Creator接口是为了完成反序列化。

看一个例子

``` java
public class MyParcelable implements Parcelable
{
     private int mData;

     public int describeContents()
     {
         return 0;
     }

     public void writeToParcel(Parcel out, int flags)
     {
         out.writeInt(mData);
     }

     public static final Parcelable.Creator<MyParcelable> CREATOR = new Parcelable.Creator<MyParcelable>()
     {
         public MyParcelable createFromParcel(Parcel in)
         {
             return new MyParcelable(in);
         }

         public MyParcelable[] newArray(int size)
         {
             return new MyParcelable[size];
         }
     };

     private MyParcelable(Parcel in)
     {
         mData = in.readInt();
     }
 }
```

这一个类只有一个成员属性mData。我们想把这个类实现Parcelable接口。

writeToParcel方法，把一个个成员对象**按顺序**写入
包裹中。

注意类中还有一个新的私有构造方法，参数是一个Parcel。这里是为了完成反序列化，从包裹中**按顺序**完成对象构造。**所以这里序列化顺序和反序列化顺序都不能出错！**


实现Creator接口，这里决定如何返回一个反序列化对象直接用parcel直接构造一个新对象返回即可。


简而言之，通过writeToParcel将你的对象映射成Parcel对象，再通过createFromParcel将Parcel对象映射成你的对象。也可以将Parcel看成是一个流，通过writeToParcel把对象写到流里面，在通过createFromParcel从流里读取对象，只不过这个过程需要你来实现，因此写的顺序和读的顺序必须一致。
