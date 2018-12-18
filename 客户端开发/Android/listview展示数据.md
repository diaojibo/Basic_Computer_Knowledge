## listView
在android开发中ListView是比较常用的组件，它以列表的形式展示具体内容，并且能够根据数据的长度自适应显示。

列表的显示需要三个元素：

1. ListVeiw 用来展示列表的View。

2. 适配器 用来把数据映射到ListView上的中介。

3. 数据    具体的将被映射的字符串，图片，或者基本组件。

根据列表的适配器类型，列表分为三种，ArrayAdapter，SimpleAdapter和SimpleCursorAdapter

其中以ArrayAdapter最为简单，只能展示一行字。SimpleAdapter有最好的扩充性，可以自定义出各种效果。SimpleCursorAdapter可以认为是SimpleAdapter对数据库的简单结合，可以方面的把数据库的内容以列表的形式展示出来。


### 一个简单例子
xml中加入一个listView控件。

``` xml
<?xmlversion="1.0"encoding="utf-8"?>
<LinearLayoutxmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    >
<!-- 添加一个ListView控件 -->
<ListView
    android:id="@+id/lv"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    />
</LinearLayout>
```

``` java
public class MyListView extends Activity {

private static final String[] strs = new String[] {
    "first", "second", "third", "fourth", "fifth"
    };//定义一个String数组用来显示ListView的内容
private ListView lv;

/** Called when the activity is first created. */
@Override
public void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

lv = (ListView) findViewById(R.id.lv);//得到ListView对象的引用
/*为ListView设置Adapter来绑定数据*/
lv.setAdapter(new ArrayAdapter<String>(this,
                android.R.layout.simple_list_item_1, strs));

    }
}
```

结果而言，无非是创建了一个数组，然后把这个数组用于创建adapter再把adapter传进去。
