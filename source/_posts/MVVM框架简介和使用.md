---
title: MVVM框架简介和使用
date: 2016-09-22 16:09:18
categories:
- Android技术
tags:
- 框架
---

### MVVM简介

MVVM是由MVC-->MVP-->MVVM一步步进化来的，那么先让我们了解下：MVC和MVP

#### MVC
MVC 即模型——视图——控制器。这样设计使得程序的各个部分分离降低耦合性，我们对代码的结构进行了划分。

- 视图（View）：用户界面。
- 控制器（Controller）：业务逻辑。
- 模型（Model）：数据保存。

<img src="/img/android/mvvm/mvc.jpg" />

大概流程是，View传送指令到Controller，Controller完成业务逻辑后，要求Model改变状态，Model将新的数据发送到View，用户得到反馈。


#### MVP
<img src="/img/android/mvvm/mvp.jpg" />

Presenter 替换掉了Controller，不仅仅处理逻辑部分。而且还控制着View的刷新，监听Model层的数据变化。这样隔离掉View和Model的关系后使得View层变的非常的薄，没有任何的逻辑部分又不用主动监听数据，被称之为“被动视图”。

#### MVVM
<img src="/img/android/mvvm/mvvm.jpg" />

至于MVVM基本上和MVP一模一样，感觉只是名字替换了一下。他的关键技术就是今天的主题(Data Binding)。View的变化可以自动的反应在ViewModel，ViewModel的数据变化也会自动反应到View上。这样开发者就不用处理接收事件和View更新的工作，框架已经帮你做好了。

### Data Binding Library

去年的Google IO 大会上，Android 团队发布了一个数据绑定框架（Data Binding Library）。以后可以直接在 layout 布局 xml 文件中绑定数据了，无需再 findViewById 然后手工设置数据了。其语法和使用方式和 JSP 中的 EL 表达式非常类似。 下面就来介绍怎么使用Data Binding Library。

#### 基本使用
目前，最新版的Android Studio已经内置了该框架的支持，配置起来也很简单，只需要编辑app目录下的build.gradle文件，添加下面的内容就好了

```
android {
    ....
    dataBinding {
        enabled = true
    }
}


```

要使用数据绑定，我们得首先创建一个实体类，比如User实体类，如下：

```
public class UserEntity {
    private String username;
    private String nickname;
    private int age;

    public UserEntity() {
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getNickname() {
        return nickname;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public UserEntity(int age, String nickname, String username) {
        this.age = age;
        this.nickname = nickname;
        this.username = username;
    }
}

```

下面我们来看看layout文件是需要怎么书写

```
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    >

    <data>

        <variable
            name="user"
            type="org.lenve.databinding1.UserEntity"/>
    </data>

    <LinearLayout
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context="org.lenve.databinding1.MainActivity">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.username}"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.nickname}"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(user.age)}"/>
    </LinearLayout>
</layout>

```

你会发现布局文件不再是以传统的某一个容器作为根节点，而是使用<layout></layout>作为根节点，在<layout>节点中我们可以通过<data>节点来引入我们要使用的数据源。

- 在data中定义的variable节点，name属性表示变量的名称，type表示这个变量的类型，实例就是我们实体类的位置，当然，这里你也可以换一种写法，如下：

```
  <data>

        <import type="org.lenve.databinding1.UserEntity"/>
        <variable
            name="user"
            type="UserEntity"/>
    </data>

```

先使用import节点将UserEntity导入，然后直接使用即可。但是如果这样的话又会有另外一个问题，假如我有两个类都是UserEntity，这两个UserEntity分属于不同的包中，又该如何？看下面：

```
   <data>

        <import type="org.lenve.databinding1.UserEntity" alias="CentMeng"/>
        <variable
            name="user"
            type="CentMeng"/>
    </data>

```

在import节点中还有一个属性叫做alias，这个属性表示我可以给该类取一个别名，我给UserEntity这个实体类取一个别名叫做Lenve，这样我就可以在variable节点中直接写CentMeng了。

看完data节点我们再来看看布局文件，TextView的text属性被我直接设置为了@{user.username},这样，该TextView一会直接将UserEntity实体类的username属性的值显示出来，对于显示age的TextView，我用了String.valueOf来显示，因为大家知道TextView并不能直接显示int型数据，所以需要一个简单的转换，事实上，我们还可以在{}里边进行一些简单的运算，这些我一会再说。

最后，我们来看看Activity中该怎么写，setContentView方法不能够再像以前那样来写了，换成下面的方式：

```
DataBindingUtil.setContentView(this, R.layout.activity_main)

```

该方法有一个返回值，这个返回值就是系统根据我们的activity_main.xml布局生成的一个ViewModel类，所以完整写法如下：

```

ActivityMainBinding activityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);

```

有了ViewModel，再把数据绑定上去就可以了，如下：

```
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityMainBinding activityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        UserEntity user = new UserEntity();
        user.setAge(34);
        user.setUsername("zhangsan");
        user.setNickname("张三");
        activityMainBinding.setUser(user);
    }

```

#### 基本的运算

##### 基本三目运算

```
  <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{`username is :`+user.username}"/>


```

两个??表示如果username属性为null则显示nickname属性，否则显示username属性。


##### 字符拼接
```
<TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{`username is :`+user.username}"/>        
```
     
***这里的字符拼接不是用单引号哦，用的是ESC按键下面那个按键按出来的。目前DataBinding中的字符拼接还不支持中文。***


##### 根据数据来决定显示样式
 
```
<TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@{user.age &lt; 24 ? 0xFF0000FF:0xFFFF0000}"
            android:text="@{String.valueOf(user.age)}"/>
```
我在这里给TextView设置背景的时候，做了一个简单的判断，如果用户的年龄小于24，背景就显示为蓝色，否则背景就显示为红色，DataBinding里支持小于号但是不支持大于号，索性，大于小于号我都用转义字符来表示。


##### 其他运算

基本四则运算，逻辑与、逻辑或、取反位移等

- 数学表达式 + – / * %

- 字符串链接 +

- 逻辑操作符 && ||

- 二元操作符 & | ^

- 一元操作符 + – ! ~

- Shift >> >>> <<

- 比较 == > < >= <=< p="">
- instanceof

- Grouping ()

- Literals – character, String, numeric, null

- 值域引用（Field access）

- 通过[]访问数组里面的对象


#### 自定义绑定（ImageView绑定示例）

这里我们还得介绍DataBinding的另一项新功能，就是关于DataBinding自定义属性的问题，事实上，在我们使用DataBinding的时候，可以给一个控件自定义一个属性，比如我们下面即将说的这个绑定ImageView的案例。假设我现在想要通过Picasso显示一张网络图片，正常情况下这个显示很简单，可是如果我要通过DataBinding来实现，该怎么做呢？我们可以使用

```
@BindingAdapter
```


注解来创建一个自定义属性，同时还要有一个配套的注解的方法。当我们在布局文件中使用这个自定义属性的时候，会触发这个被我们注解的方法，这样说大家可能还有一点模糊，我们来看看新的实体类：


```
public class User {
    private String username;
    private String userface;

    public User() {
    }

    public User(String userface, String username) {
        this.userface = userface;
        this.username = username;
    }

    @BindingAdapter("bind:userface")
    public static void getInternetImage(ImageView iv, String userface) {
        Picasso.with(iv.getContext()).load(userface).into(iv);
    }

    public String getUserface() {
        return userface;
    }

    public void setUserface(String userface) {
        this.userface = userface;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
```

新类里边只有两个属性，分别是用户名和用户图像，用户图像中存储的实际上是一个网络图片地址，这里除了基本的get/set方法之外还多了一个叫做getInternetImage的网络方法，这个方法有一个注解@BindAdapter("bind:userface")，该注解表示当用户在ImageView中使用自定义属性userface的时候，会触发这个方法，我在这个方法中来为这个ImageView加载一张图片，这里有一点需要注意，就是该方法必须为静态方法。OK，我们再来看看这次的布局文件：

```
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    >

    <data>

        <variable
            name="user"
            type="org.lenve.databinding2.User"/>
    </data>

    <LinearLayout
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context="org.lenve.databinding2.MainActivity">

        <ImageView
            android:id="@+id/iv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:userface="@{user.userface}"></ImageView>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.username}"/>
    </LinearLayout>
</layout>
```

***在ImageView控件中使用userface属性的时候，使用的前缀不是android而是app。***再来看看Activity中的代码：

```
   @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityMainBinding dataBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        dataBinding.setUser(new User("http://img2.cache.netease.com/auto/2016/7/28/201607282215432cd8a.jpg", "张三"));
    }
```

#### 绑定ListView

现在google退出了RecyclerView，其实他和ListView的方式是一样的，通用的。

下面是主体布局和item布局

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="org.lenve.databinding3.MainActivity">

    <ListView
        android:id="@+id/lv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"></ListView>
</RelativeLayout>
```

```
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    >

    <data>

        <variable
            name="food"
            type="org.lenve.databinding3.Food"/>
    </data>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="96dp"
        android:orientation="vertical">

        <ImageView
            android:id="@+id/iv"
            android:layout_width="96dp"
            android:layout_height="96dp"
            android:padding="6dp"
            app:img="@{food.img}"/>

        <TextView
            android:id="@+id/description"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="8dp"
            android:layout_toRightOf="@id/iv"
            android:ellipsize="end"
            android:maxLines="3"
            android:text="@{food.description}"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="8dp"
            android:layout_toRightOf="@id/iv"
            android:layout_alignParentBottom="true"
            android:layout_marginBottom="2dp"
            android:text="@{food.keywords}"
            android:textStyle="bold"/>
    </RelativeLayout>
</layout>
```

其他的和上面介绍一样，这里就不粘贴代码了，我们主要看看adapter吧

```
public class MyBaseAdapter<T> extends BaseAdapter {
    private Context context;
    private LayoutInflater inflater;
    private int layoutId;
    private int variableId;
    private List<T> list;

    public MyBaseAdapter(Context context, int layoutId, List<T> list, int resId) {
        this.context = context;
        this.layoutId = layoutId;
        this.list = list;
        this.variableId = resId;
        inflater = LayoutInflater.from(context);
    }

    @Override

    public int getCount() {
        return list.size();
    }

    @Override
    public Object getItem(int position) {
        return list.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewDataBinding dataBinding;
        if (convertView == null) {
            dataBinding = DataBindingUtil.inflate(inflater, layoutId, parent, false);
        }else{
            dataBinding = DataBindingUtil.getBinding(convertView);
        }
        dataBinding.setVariable(variableId, list.get(position));
        return dataBinding.getRoot();
    }
}
```

这个大概算是Adapter的终极写法了，如果你按这种方式来写Adapter，那么如果没有非常奇葩的需求，你这个App中可能就只有这一个给ListView使用的Adapter了，为什么这么说呢？因为这个Adapter中没有一个变量和我们的ListView沾边，解释一下几个变量吧：layoutId这个表示item布局的资源id，variableId是系统自动生成的，根据我们的实体类，直接从外部传入即可。另外注意布局加载方式为DataBindingUtil类中的inflate方法。OK，最后再来看看Activity:


```
public class MainActivity extends AppCompatActivity {

    private Handler mHandler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            MyBaseAdapter<Food> adapter = new MyBaseAdapter<>(MainActivity.this, R.layout.listview_item, foods, org.lenve.databinding3.BR.food);
            lv.setAdapter(adapter);
        }
    };
    private List<Food> foods;
    private ListView lv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        lv = ((ListView) findViewById(R.id.lv));
        initData();
    }

    private void initData() {
        OkHttpClient client = new OkHttpClient.Builder().build();
        Request request = new Request.Builder().url("http://www.tngou.net/api/food/list?id=1").build();
        client.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {

            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                if (response.isSuccessful()) {
                    parseJson(response.body().string());
                }
            }
        });
    }

    private void parseJson(String jsonStr) {
        foods = new ArrayList<>();
        try {
            JSONObject jo = new JSONObject(jsonStr);
            JSONArray tngou = jo.getJSONArray("tngou");
            for (int i = 0; i < tngou.length(); i++) {
                JSONObject item = tngou.getJSONObject(i);
                String description = item.getString("description");
                String img = "http://tnfs.tngou.net/image"+item.getString("img");
                String keywords = "【关键词】 "+item.getString("keywords");
                String summary = item.getString("summary");
                foods.add(new Food(description, img, keywords, summary));
            }
            mHandler.sendEmptyMessage(0);
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
}
```

#### 点击事件处理

如果你使用DataBinding，我们的点击事件也会有新的处理方式，首先以ListView为例来说说如何绑定点击事件，在listview_item布局文件中每一个item的根节点添加如下代码：

```
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    >
	....
	....
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="96dp"
        android:onClick="@{food.onItemClick}"
        android:orientation="vertical">

        <ImageView
            android:id="@+id/iv"
            android:layout_width="96dp"
            android:layout_height="96dp"
            android:padding="6dp"
            app:img="@{food.img}"/>
		....
		....
		....
    </RelativeLayout>
</layout>
```

OK，我给RelativeLayout容器添了onClick属性，属性的值为food.onItemClick，那么这个onItemClick到底是什么呢？其实就是在实体类Food中定义的一个方法，如下：

```
 public void onItemClick(View view) {
        Toast.makeText(view.getContext(), getDescription(), Toast.LENGTH_SHORT).show();
    }
```

点击item获取当前position的数据，获取方式也是非常简单，直接get方法获取即可，比传统的ListView的点击事件通过position来获取数据方便多了。如果我想为关键字这个TextView添加点击事件也很简单，和上面一样，这里我就不再贴代码了.

#### 数据更新处理

单纯的更新Food对象并不能改变ListView的UI显示效果，那该怎么做呢？Google给我们提供了三种解决方案，分别如下：

##### 让实体类继承自BaseObservable (最常用)

让实体类继承自BaseObservable，然后给需要改变的字段的get方法添加上@Bindable注解，然后给需要改变的字段的set方法加上notifyPropertyChanged(org.lenve.databinding3.BR.description);一句即可，比如我想点击item的时候把description字段的数据全部改为111，我可以修改Food类变为下面的样子：

```
public class Food extends BaseObservable {
    private String description;
    private String img;
    private String keywords;
    private String summary;

    public Food() {
    }

    public Food(String description, String img, String keywords, String summary) {
        this.description = description;
        this.img = img;
        this.keywords = keywords;
        this.summary = summary;
    }

    @BindingAdapter("bind:img")
    public static void loadInternetImage(ImageView iv, String img) {
        Picasso.with(iv.getContext()).load(img).into(iv);
    }

    public void onItemClick(View view) {
//        Toast.makeText(view.getContext(), getDescription(), Toast.LENGTH_SHORT).show();
        setDescription("111");
    }

    public void clickKeywords(View view) {
        Toast.makeText(view.getContext(), getKeywords(), Toast.LENGTH_SHORT).show();
    }


    @Bindable
    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
        notifyPropertyChanged(org.lenve.databinding3.BR.description);
    }

    public String getImg() {
        return img;
    }

    public void setImg(String img) {
        this.img = img;
    }

    public String getKeywords() {
        return keywords;
    }

    public void setKeywords(String keywords) {
        this.keywords = keywords;
    }

    public String getSummary() {
        return summary;
    }

    public void setSummary(String summary) {
        this.summary = summary;
    }
}
```

#### 使用DataBinding提供的ObservableFields来创建实体类

这种方式使用起来略微麻烦，除了继承BaseObservable之外，创建属性的方式也变成下面这种：

```
private final ObservableField<String> description = new ObservableField<>();
```
属性的读写方式也变了，读取方式如下：


```
description.get()
```

写入方式如下：

```
this.description.set(description);
```

OK，依据上面几个规则，我新定义的实体类如下：

```
public class Food extends BaseObservable {
    private final ObservableField<String> description = new ObservableField<>();
    private final ObservableField<String> img = new ObservableField<>();
    private final ObservableField<String> keywords = new ObservableField<>();
    private final ObservableField<String> summary = new ObservableField<>();
    public Food() {
    }

    public Food(String description, String img, String keywords, String summary) {
        this.description.set(description);
        this.keywords.set(keywords);
        this.img.set(img);
        this.summary.set(summary);
    }

    @BindingAdapter("bind:img")
    public static void loadInternetImage(ImageView iv, String img) {
        Picasso.with(iv.getContext()).load(img).into(iv);
    }

    public void onItemClick(View view) {
//        Toast.makeText(view.getContext(), getDescription(), Toast.LENGTH_SHORT).show();
        setDescription("111");
    }

    public void clickKeywords(View view) {
        Toast.makeText(view.getContext(), getKeywords(), Toast.LENGTH_SHORT).show();
    }


    @Bindable
    public String getDescription() {
        return description.get();
    }

    public void setDescription(String description) {
        this.description.set(description);
        notifyPropertyChanged(org.lenve.databinding3.BR.description);
    }

    public String getImg() {
        return img.get();
    }

    public void setImg(String img) {
        this.img.set(img);
    }

    public String getKeywords() {
        return keywords.get();
    }

    public void setKeywords(String keywords) {
        this.keywords.set(keywords);
    }

    public String getSummary() {
        return summary.get();
    }

    public void setSummary(String summary) {
        this.summary.set(summary);
    }
}
```

#### 使用DataBinding中提供的集合来存储数据即可

DataBinding中给我们提供了一些现成的集合，用来存储数据，比如ObservableArrayList，ObservableArrayMap，因为这些用的少，我这里就不做介绍了。
