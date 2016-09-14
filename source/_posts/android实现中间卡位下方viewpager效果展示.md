---
title: android实现中间卡位下方viewpager效果展示
date: 2016-07-25 19:11:35
categories:
- Android技术
tags:
- 布局样式
---

<img src="/img/data/androidkawei.gif" />

###设置卡位效果###

实现如图效果，中间tab滑动上方时候，卡在顶部，下方继续上滑

其实很简单，只需要借助android的[Design Support Library](https://guides.codepath.com/android/Design-Support-Library)

####1.引用library
```
 compile 'com.android.support:design:23.3.0'
 ```
 
 ####2.写布局
 
 ```
  <android.support.design.widget.CoordinatorLayout
            android:id="@+id/coordinator"
            android:layout_marginTop="@dimen/view_height"
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <android.support.design.widget.AppBarLayout
                android:id="@+id/barlayout"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_above="@+id/layout_zhidian"
                android:background="@color/background"
                android:theme="@style/ThemeOverlay.AppCompat.Dark">

                <FrameLayout
                    android:id="@+id/layout_topexpert"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:background="@color/white"
                    app:layout_scrollFlags="scroll|exitUntilCollapsed"
                    android:visibility="visible">

                  <....>
                  <..../>
                </FrameLayout>

                <android.support.design.widget.TabLayout
                    android:id="@+id/tl_expert"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    app:tabGravity="center"
                    app:tabIndicatorColor="@color/basecolor"
                    android:background="@color/white"
                    app:tabMode="fixed"
                    android:layout_gravity="center"
                    app:tabSelectedTextColor="@color/basecolor"
                    app:tabTextColor="@color/darkgray"
                    />
            </android.support.design.widget.AppBarLayout>

            <android.support.v4.view.ViewPager
                android:id="@+id/viewpager"
                app:layout_behavior="@string/appbar_scrolling_view_behavior"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />

        </android.support.design.widget.CoordinatorLayout>
        
 ```
 
 其中关键的是[CoordinatorLayout](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html)
 
 
#####设置滑动布局#####

 
 在CoordinatorLayout中包含AppBarLayout,其中AppBarLayout中的子布局，如果设置 
app:layout_scrollFlags属性里面必须至少启用scroll这个flag，这样这个view才会滚动出屏幕，否则它将一直固定在顶部。可以使用的其他flag有：

***enterAlways: 一旦向上滚动这个view就可见。***

***enterAlwaysCollapsed: 顾名思义，这个flag定义的是何时进入（已经消失之后何时再次显示）。假设你定义了一个最小高度（minHeight）同时enterAlways也定义了，那么view将在到达这个最小高度的时候开始显示，并且从这个时候开始慢慢展开，当滚动到顶部的时候展开完。***

***exitUntilCollapsed: 同样顾名思义，这个flag时定义何时退出，当你定义了一个minHeight，这个view将在滚动到达这个最小高度的时候消失。***

***记住，要把带有scroll flag的view放在前面，这样收回的view才能让正常退出，而固定的view继续留在顶部。***

本例中layout_topexpert是使用滑动消失的设置了 app:layout_scrollFlags="scroll|exitUntilCollapsed"，而tl_expert是卡在上方的，所以没有做任何设置

#####定义AppBarLayout与滚动视图之间的联系#####
在viewpager或者任意支持嵌套滚动的view比如NestedScrollView上添加app:layout_behavior。support library包含了一个特殊的字符串资源@string/appbar_scrolling_view_behavior，它和AppBarLayout.ScrollingViewBehavior相匹配，用来通知AppBarLayout 这个特殊的view何时发生了滚动事件，这个behavior需要设置在触发事件（滚动）的view之上

*****另外需要注意的是*****

如果相关滚动的是viewpager，在viewpager可以使用recyclerview和scrollview，但ScrollView要使用NestedScrollView



<br/>

-----
<br/>
滑动卡位的效果实现了，下面是实现Tabfenye和viewpager下方的效果了，其实也很简单，使用库中的android.support.design.widget.TabLayout并做些设置就ok了

###设置Tablayout###

####1.设置ViewPager的adpter

代码很简单，直接看代码

```

public class ExpertDetailPagerAdapter extends FragmentPagerAdapter {

    private String userId = "";

    private String[] tabTitles = new String[]{"点师介绍","话题","益答","动态"};


    @Override
    public CharSequence getPageTitle(int position) {
        return tabTitles[position];
    }

    public ExpertDetailPagerAdapter(FragmentManager fm, String userId) {
        super(fm);
        this.userId = userId;
    }

    @Override
    public int getCount() {
       return 4;
    }

    @Override
    public Fragment getItem(int position) {
        switch (position){
            case 0:
                return ExpertInfoFragment.newInstance(userId);
            case 1:
                return ExpertTopicFragment.newInstance(userId);
            case 2:
                RankEntity entity = new RankEntity();
                entity.setId(userId);
                return ExpertYiDaFragment.newInstance(entity);
            case 3:
                return ExpertDynamicFragment.newInstance(userId);
        }
        return null;

    }
 }   

```
getPageTitle是返回Tab的标题

####2.设置Tablayout

```
 fragmentAdapter = new ExpertDetailPagerAdapter(getSupportFragmentManager(),expertId);
  viewpager.setAdapter(fragmentAdapter);
  viewpager.setOffscreenPageLimit(3);

  tl_expert.setupWithViewPager(viewpager);


```

本例中Tablayout是居中显示，是通过在布局中设置这两个属性实现的

```

	app:tabGravity="center"
	app:tabMode="fixed"
           
```
 

这样就实现了，我一开始使用ScrollView嵌套ViewPager，ViewPager中嵌套ScrollView，RecyclerView出现一大堆问题，最后发现用design库如此的简单解决了问题



###自定义Behavior###


CoordinatorLayout的工作原理是搜索定义了[CoordinatorLayout Behavior](http://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html) 的子view，不管是通过在xml中使用app:layout_behavior标签还是通过在代码中对view类使用@DefaultBehavior修饰符来添加注解。当滚动发生的时候，CoordinatorLayout会尝试触发那些声明了依赖的子view。

要自己定义CoordinatorLayout Behavior，你需要实现layoutDependsOn() 和onDependentViewChanged()两个方法。比如AppBarLayout.Behavior 就定义了这两个关键方法。这个behavior用于当滚动发生的时候让AppBarLayout发生改变。

```

public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency) {
    return dependency instanceof AppBarLayout;
}

public boolean onDependentViewChanged(CoordinatorLayout parent, View child, View dependency) {
    // check the behavior triggered
    android.support.design.widget.CoordinatorLayout.Behavior behavior = ((android.support.design.widget.CoordinatorLayout.LayoutParams) dependency.getLayoutParams()).getBehavior();
    if (behavior instanceof AppBarLayout.Behavior) {
        // do stuff here
    }
}

```