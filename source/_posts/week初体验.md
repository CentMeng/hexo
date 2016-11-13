---
title: Weex初体验
date: 2016-09-28 17:39:31
categories:
- 前段技术
tags:
- Weex
---

# Weex 初体验
在Weex中坑了几天，终于在Android中跑出了第一个demo，此demo会不断的更新。

Git地址：[https://github.com/CentMeng/WeexDemo](https://github.com/CentMeng/WeexDemo)
## 搭建相关
### app/build.gradle
```java
   compile 'com.taobao.android:weex_sdk:0.8.0+'
    compile 'com.alibaba:fastjson:1.1.46.android'
    compile 'com.github.bumptech.glide:glide:3.7.0'
```

### Activity
*Activity也可以设置*

*ImageAdaptermInstance.setImgLoaderAdapter(new ImageAdapter(this));*

```java
@EActivity(R.layout.ac_main)
public class MainActivity extends AppCompatActivity {

    @ViewById
    FrameLayout container;
    private WXSDKInstance mWeexInstance;

    @AfterViews
    void afterView(){
        // sdk 实例
        mWeexInstance = new WXSDKInstance(this);
        mWeexInstance.registerRenderListener(new IWXRenderListener() {

            // sdk 将 js 文件渲染成 view 对象回调
            @Override
            public void onViewCreated(WXSDKInstance wxsdkInstance, View view) {
                if (container != null) {
                    container.addView(view); // 添加到界面
                }
            }

            @Override
            public void onRenderSuccess(WXSDKInstance wxsdkInstance, int i, int i1) {

            }

            @Override
            public void onRefreshSuccess(WXSDKInstance wxsdkInstance, int i, int i1) {

            }

            @Override
            public void onException(WXSDKInstance wxsdkInstance, String s, String s1) {

            }
        });
        // 加载 js 文件
        mWeexInstance.render("001",
                WXFileUtils.loadAsset("build.js", this),
                null, null, -1, -1,
                WXRenderStrategy.APPEND_ASYNC);
    }

}

```

### Application
```java
public class WXApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        //代码中启动Weex RunTime，用于渲染UI
        WXEnvironment.addCustomOptions("appName","TBSample");
        InitConfig config = new InitConfig.Builder()
                .setImgAdapter(new ImageAdapter())
                .build();
        WXSDKEngine.initialize(this, config);
    }
}

```

### ImageAdapter，用Glide加载图片
```java

/**
 * @author Vincent.M
 * @date 16/9/28
 * @copyright ©2016 孟祥程 All Rights Reserved
 * @desc Weex 加载图片
 */
public class ImageAdapter implements IWXImgLoaderAdapter {
    @Override
    public void setImage(final String url, final ImageView view, WXImageQuality quality, WXImageStrategy strategy) {
        WXSDKManager.getInstance().postOnUiThread(new Runnable() {

            @Override
            public void run() {
                //.dontAnimate()是为了解决圆形图片第一次加载不出来只显示展位图
                Glide.with(WXEnvironment.getApplication()).load(url).placeholder(R.drawable.bg_notus).crossFade().dontAnimate().into(view);
            }
        }, 0);
    }
}
```


## 版本记录
### V0.1 环境搭建，运行demo