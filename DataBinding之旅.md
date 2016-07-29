### 一.module中的配置：引入dataBinding功能

```
//在android{}中
dataBinding {
        enabled true
}
```
### 二.布局文件格式

```
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data class=".UserBinding">
        //数据绑定
        <variable
            name="user"
            type="cn.yjjc.com.mydatabindingtest.User"/>
        <variable
            name="num"
            type="Integer"/>
    </data>
    <RelativeLayout>
        //布局内容
    </RelativeLayout>
</layout>
```
### 三.获取dataBinding

```
UserBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

//也可以用Activity自动生成的binding(ActivityNameBinding)，但需要注意：
当编译器版本为24的时候（最新版本），Activity自动生成的activityNamebinding无法编译。两种解决方案：
1.降级buildTool_version到23
2.使用自定义class，如<data class=".MyDefineBinding>....
```
### 四.在布局中绑定数据

```
//注意：@{...}表达式中不能包含中文
<TextView
    android:id="@+id/tv_name"
    android:text='@{"name: " + user.name}'
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />

<TextView
    android:id="@+id/tv_age"
    android:layout_below="@id/tv_name"
    android:text='@{"age: " + user.age}'
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

### 五.获取View和绑定的数据
- TextView tv = binding.tv;
- User user = Binding.user;
### 六.绑定图片地址（自动加载图片）
###### 1. 在与布局绑定的数据Model中加入

```
public class User {
    private String name;
    private int age;
    private String imageUrl;
    //Getter and Setter
    //1.从布局中取出error图片(‘imageUrl’必须和属性名imageUrl保持一致)
    @BindingAdapter({"imageUrl", "error"})
    public static void loadImage(ImageView view, String imageUrl, Drawable error) {
        Picasso.with(view.getContext()).load(imageUrl).error(error).into(view);
    }
    
    //2.直接将error图片写在代码中
    @BindingAdapter({"imageUrl"})
    public static void loadImage(ImageView view, String imageUrl) {
        Picasso.with(view.getContext()).load(imageUrl).error(R.mipmap.ic_launcher).into(view);
    }
}
```
###### 2.在布局中

```
//1.将error图片写在布局中
//注意：error只支持drawable，不支持mipmap
<ImageView
    android:id="@+id/iv"
    imageUrl="@{user.imageUrl}"
    error="@{@drawable/bluepl}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
//2.将error图片写在代码中
<ImageView
    android:layout_width="200dp"
    android:layout_height="200dp"
    imageUrl="@{viewModel.imageUrl}" />
```
### 七.使用include的标签，并将数据传到子布局
###### 1.子布局layout_user.xml:
```
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="user"
            type="cn.yjjc.com.mydatabindingtest.User"/>
    </data>
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <TextView
            android:text="@{user.name}"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
        <TextView
            android:text="@{String.valueOf(user.age)}"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
    </LinearLayout>
</layout>
```
###### 2.在主布局中导入include 标签(include标签支持嵌套布局)

```
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data class=".UserBinding">
        <variable
            name="user"
            type="cn.yjjc.com.mydatabindingtest.User"/>
    </data>
    <RelativeLayout>
        //....
        <include
            android:id="@+id/layout"
            layout="@layout/layout_user"
            app:user="@{user}"
            />
        <LinearLayout
            android:id="@+id/ll_layout"
            android:layout_below="@id/layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <include
                android:id="@+id/layout"
                layout="@layout/layout_user"
                user="@{user}"
                />
        </LinearLayout>
    </RelativeLayout>
</layout>
```
### 八.更新数据
##### 1.定义：任何 POJO 对象都能用在 Data Binding 中，但是更改 POJO 并不会同步更新 UI。Data Binding 的强大之处就在于它可以让你的数据拥有更新通知的能力。
##### 2.绑定数据到xml布局中，这样数据改变后会自动通知xml布局更新，有三种方式
###### ①Observable 对象

```
//1.实现POJO对象，继承于BaseObservable
public class ObservableUser extends BaseObservable {
    private String firstName;
    private String lastName;

    @Bindable
    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
        notifyPropertyChanged(BR.firstName);
    }

    @Bindable
    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
        notifyPropertyChanged(BR.lastName);
    }
}
//2.在布局文件中->data标签中绑定数据，在视图中添加控件
<variable
    name="user"
    type="cn.yjjc.com.mydatabindingtest.observable.ObservableUser"/>
<TextView
    android:text="@{user1.name}"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
<TextView
    android:text="@{String.valueOf(user1.age)}"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
//3.在Activity中，绑定数据
private ObservableUser user = new ObservableUser();
ActivityObservableUserBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_observable_user);
binding.setUser(user);
//4.在Activity改变数据，xml视图中会自动改变
user.setFirstName("FirstName1");
user.setLastName("LastName1");
```

###### ②ObservableFields

```
//1.在Activity中或POJO中声明ObservableFields
private ObservableInt varNum = new ObservableInt(7);
private ObservableField<String> varName = new ObservableField<>("originName");
//2.在布局文件中->data标签中绑定数据，在视图中添加控件
<variable
    name="varNum"
    type="android.databinding.ObservableInt"/>
<variable
    name="varName"
    type="ObservableField&lt;String>"/>
<TextView
    android:text="@{String.valueOf(varNum.get())}"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
<TextView
    android:text="@{varName.get()}"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
//3.在Activity中，绑定数据
binding.setVarNum(varNum);
binding.setVarName(varName);
//4.在Activity改变数据，xml视图中会自动改变
varNum.set(17);
varName.set("Name1");
```

###### ③Observable Collections 容器类

```
//1.在Activity中或POJO中声明ObservableFields
private ObservableMap<String, Object> varMap = new ObservableArrayMap<>();
private ObservableList<String> varList = new ObservableArrayList<>();
//2.在布局文件中->data标签中绑定数据，在视图中添加控件
<variable
    name="varMap"
    type="ObservableMap&lt;String, Object>"/>
<variable
    name="varList"
    type="android.databinding.ObservableList&lt;String>"/>
<TextView
    android:text='@{varMap["name"]}'
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
<TextView
    android:text='@{String.valueOf(varMap["age"])}'
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
<TextView
    android:text='@{varList[0]}'
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
<TextView
    android:text='@{varList[1]}'
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
//3.在Activity中，绑定数据
binding.setVarMap(varMap);
binding.setVarList(varList);
//4.在Activity改变数据，xml视图中会自动改变
varMap.put("name", "chenlong");
varMap.put("age", 17);

varList.clear();
varList.add("chenlong1");
varList.add("chenlong2");
```
### 九.向ListView中填充数据
##### 1.写Activity的布局xml

```
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
    </data>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <ListView
            android:id="@+id/mListView"
            android:layout_width="match_parent"
            android:layout_weight="wrap_content"/>
        <Button
            android:text="Button"
            android:onClick="onClick"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
    </LinearLayout>
</layout>
```
##### 2.写Adapter的Item布局xml

```
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="value"
            type="String"/>
    </data>
    <TextView
        android:padding="20dip"
        android:text="@{value}"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</layout>
```

##### 3.写Adapter

```
public class StringAdapter extends BaseAdapter {
    private List<String> list;
    private LayoutInflater mInflater;

    public StringAdapter(Context context, List<String> list) {
        mInflater = LayoutInflater.from(context);
        this.list = list;
    }

    @Override
    public int getCount() {
        return list==null?0:list.size();
    }

    @Override
    public String getItem(int position) {
        return list.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        LayoutItemBinding binding;
        if (convertView == null) {
            convertView = mInflater.inflate(R.layout.layout_item, parent, false);
            //得到与convertView绑定的DataBinding
            binding = DataBindingUtil.bind(convertView);
            convertView.setTag(binding);
        } else {
            binding = (LayoutItemBinding) convertView.getTag();
        }
        binding.setValue(list.get(position));
        //binding.executePendingBindings();//不加也可以更新，暂不知道它的作用

        return convertView;
    }
}
```

##### 4.写Activity中的代码

```
private List<String> list = new ArrayList<>();
private ActivityListBinding binding;
private StringAdapter adapter1;
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    binding = DataBindingUtil.setContentView(this, R.layout.activity_list);

    adapter1 = new StringAdapter(this, list);
    adapter2 = new RecyclerAdapter(list);
    binding.mListView.setAdapter(adapter1);
}
//点击按钮更新数据
public void onClick(View view) {
    list.add("item1");
    list.add("item2");
    list.add("item3");
    list.add("item4");
    list.add("item5");
    list.add("item6");
    adapter1.notifyDataSetChanged();
}
```
### 十.向RecyclerView填充数据
##### 1.写Activity的布局xml

```
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
    </data>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <Button
            android:text="Button"
            android:onClick="onClick"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
        <android.support.v7.widget.RecyclerView
            android:id="@+id/mRecyclerView"
            android:layout_width="match_parent"
            android:layout_weight="wrap_content"/>
    </LinearLayout>
</layout>
```

##### 2.写Adapter的Item布局xml

```
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="value"
            type="String"/>
    </data>
    <TextView
        android:padding="20dip"
        android:text="@{value}"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</layout>
```

##### 3.写Adapter

```
public class RecyclerAdapter extends RecyclerView.Adapter<RecyclerAdapter.StringHolder> {
    private List<String> list;
    public RecyclerAdapter(List<String> list) {
        this.list = list;
    }

    public void setList(List<String> list) {
        this.list = list;
    }

    @Override
    public StringHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        LayoutItemBinding binding = DataBindingUtil.inflate(LayoutInflater.from(parent.getContext()),
                R.layout.layout_item, parent, false);
        StringHolder holder = new StringHolder(binding.getRoot());
        holder.setBinding(binding);
        return holder;
    }

    @Override
    public void onBindViewHolder(StringHolder holder, int position) {
        // 动态绑定变量
        holder.getBinding().setValue(list.get(position));
//        holder.getBinding().executePendingBindings();//update ui
    }

    @Override
    public int getItemCount() {
        return list==null?0:list.size();
    }

    public static class StringHolder extends RecyclerView.ViewHolder {
        private LayoutItemBinding binding;
        public StringHolder(View itemView) {
            super(itemView);
            binding = DataBindingUtil.bind(itemView);
        }
        public LayoutItemBinding getBinding() {
            return binding;
        }
        public void setBinding(LayoutItemBinding binding) {
            this.binding = binding;
        }
    }
}
```

##### 4.写Activity中的代码

```
private List<String> list = new ArrayList<>();
private ActivityListBinding binding;
private RecyclerAdapter adapter2;
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    binding = DataBindingUtil.setContentView(this, R.layout.activity_list);
    
    binding.mRecyclerView.setHasFixedSize(true);
    binding.mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
    binding.mRecyclerView.setAdapter(adapter2);
}
public void onClick(View view) {
    list.add("item1");
    list.add("item2");
    list.add("item3");
    list.add("item4");
    list.add("item5");
    list.add("item6");
    adapter2.notifyDataSetChanged();
}
```
### 十一.表达式 @{}
##### 1.绑定数据来自动控制View的显隐
###### ①布局中

```
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <import type="android.view.View"/>
        <variable
            name="isHide"
            type="android.databinding.ObservableBoolean"/>
    </data>
    <LinearLayout android:orientation="vertical" android:layout_width="match_parent"
        android:layout_height="match_parent">
        <TextView
            android:visibility="@{isHide.get()?View.VISIBLE:View.GONE}"
            android:text=" I just can't wait to code my life!"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
        <Button
            android:onClick="onClick"
            android:text="Change Text Show or Hide"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
    </LinearLayout>
</layout>
```
###### ②Activity中

```
private ObservableBoolean isHide = new ObservableBoolean(true);
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityVisibleBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_visible);
        binding.setIsHide(isHide);
    }

    public void onClick(View view) {
        isHide.set(!isHide.get());
    }
```
###### ③总结重点：
- 1.<data>中导入表达式要用到的类View
- 2.android:visibility="@{isHide.get()?View.VISIBLE:View.GONE}"


### 十二.DataBinding官方文档的坑
##### 1.所有的命名空间都可以去掉
##### 2.表达式中不支持中文
##### 3.Type标签中'<'要转换为'&lt;'
##### 4.onCreate中ActivityMainBinding mBinding = DataBindingUtil.setContentView(...)突然报错，解决方案File->invalidate and reStart

### 十三.DataBinding 用于Fragment

```
public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        FragmentSiteorderBinding binding = DataBindingUtil.inflate(inflater, R.layout.fragment_siteorder, container, false);
        View view = binding.getRoot();
        return view;
    }
```





















