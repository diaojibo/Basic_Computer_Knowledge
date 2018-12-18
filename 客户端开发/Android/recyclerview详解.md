## recyclerview
跟listView一样用于展示数据，但是是个比listView先进的控件。

对比与listView的三要素(获取listview，准备好数据，设置adapter)，recyclerview的使用流程其实也和listView差别不大

``` java
mRecyclerView = findView(R.id.id_recyclerview);
//设置布局管理器
mRecyclerView.setLayoutManager(layout);
//设置adapter
mRecyclerView.setAdapter(adapter)
//设置Item增加、移除动画
mRecyclerView.setItemAnimator(new DefaultItemAnimator());
//添加分割线
mRecyclerView.addItemDecoration(new DividerItemDecoration(
                getActivity(), DividerItemDecoration.HORIZONTAL_LIST));
```

ok，相比较于ListView的代码，ListView可能只需要去设置一个adapter就能正常使用了。而RecyclerView基本需要上面一系列的步骤，那么为什么会添加这么多的步骤呢？

那么就必须解释下RecyclerView的这个名字了，从它类名上看，RecyclerView代表的意义是，我只管Recycler View，也就是说RecyclerView只管回收与复用View，其他的你可以自己去设置。可以看出其高度的解耦，给予你充分的定制自由（所以你才可以轻松的通过这个控件实现ListView,GirdView，瀑布流等效果）。

接下来我们耐心看下recyclerView的简单使用。

### 简单使用

在相对布局里放一个recyclerView

``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <android.support.v7.widget.RecyclerView
        android:id="@+id/id_recyclerview"
         android:divider="#ffff0000"
           android:dividerHeight="10dp"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</RelativeLayout>
```

然后看下作为条目的item的布局

``` xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:background="#44ff0000"
    android:layout_height="wrap_content" >

    <TextView
        android:id="@+id/id_num"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:gravity="center"
        android:text="1" />
</FrameLayout>
```

然后是java代码

``` java
public class HomeActivity extends ActionBarActivity
{

    private RecyclerView mRecyclerView;
    private List<String> mDatas;
    private HomeAdapter mAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_single_recyclerview);

        initData();
        mRecyclerView = (RecyclerView) findViewById(R.id.id_recyclerview);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        mRecyclerView.setAdapter(mAdapter = new HomeAdapter());

    }

    protected void initData()
    {
        mDatas = new ArrayList<String>();
        for (int i = 'A'; i < 'z'; i++)
        {
            mDatas.add("" + (char) i);
        }
    }

    class HomeAdapter extends RecyclerView.Adapter<HomeAdapter.MyViewHolder>
    {

        @Override
        public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
        {
            MyViewHolder holder = new MyViewHolder(LayoutInflater.from(
                    HomeActivity.this).inflate(R.layout.item_home, parent,
                    false));
            return holder;
        }

        @Override
        public void onBindViewHolder(MyViewHolder holder, int position)
        {
            holder.tv.setText(mDatas.get(position));
        }

        @Override
        public int getItemCount()
        {
            return mDatas.size();
        }

        class MyViewHolder extends ViewHolder
        {

            TextView tv;

            public MyViewHolder(View view)
            {
                super(view);
                tv = (TextView) view.findViewById(R.id.id_num);
            }
        }
    }

}
```

根据代码可以明确几点。首先，我们知道这个LayoutManager是用来指定item在整个recyclerView里的排列方式的，如果使用LinearLayout，那就如同listView一样了。

然后是adapter，这里可以知道adapter需要自己实现，写了一个类继承了`RecyclerView.Adapter`，并且其中给予了泛型参数HomeAdapter.MyViewHolder


RecyclerView.Adapter，需要实现3个方法：

#### onCreateViewHolder()

这个方法主要生成为每个Item inflater出一个View，但是该方法返回的是一个ViewHolder。该方法把View直接封装在ViewHolder中，然后我们面向的是ViewHolder这个实例，当然这个ViewHolder需要我们自己去编写。直接省去了当初的convertView.setTag(holder)和convertView.getTag()这些繁琐的步骤。

**zwlj:viewholder就是一个复用view的容器，这个方法是用来创建viewholder的，创建viewholder当然需要传入item的布局文件没然后用LayoutInflater的实例渲染就可以了。当然这个viewholder也得自己去写。**

写viewholder的时候，需要把view里的部件提取出来作为属性，这样方便复用，也就是onBindViewHolder中复用。

#### onBindViewHolder()

这个方法主要用于适配渲染数据到View中。方法提供给你了一个viewHolder，而不是原来的convertView。

**zwlj：也就是返回给你一个可以用于复用的viewholder，让你在viewholder上直接修改属性用来渲染数据。**

#### getItemCount()

这个方法就类似于BaseAdapter的getCount方法了，即总共有多少个条目。
