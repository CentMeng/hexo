---
title: Android修改字体大小效果不变
date: 2016-09-14 16:32:22
categories:
- Android技术
tags:
- 适配
---
<img src="/img/code.jpg" />

很多开发者在开发的时候，会遇到有的用户修改字体大小，从而导致适配问题，在这里，可以通过如下代码，让自己App字体大小不变。具体代码如下:

```
private void initFontScale() {
        Configuration configuration = getResources().getConfiguration();

        configuration.fontScale = (float) 1; //0.85 small size, 1 normal size, 1,15 big etc
        DisplayMetrics metrics = new DisplayMetrics();
        getWindowManager().getDefaultDisplay().getMetrics(metrics);
        metrics.scaledDensity = configuration.fontScale * metrics.density;
        getBaseContext().getResources().updateConfiguration(configuration, metrics);

```

以上代码需要每个Activity中调用，所以最好写在BaseActivity中，不喜勿喷，博主小白一个。