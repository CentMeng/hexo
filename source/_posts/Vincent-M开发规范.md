---
title: Vincent.M开发规范
date: 2016-09-20 14:25:40
categories:
- 进阶
tags:
- 规范
---
<img src="/img/code.jpg" />
### android 命名规范

#### 资源文件

图标： ic_

背景： bg_

图片: img_

资源背景命名 

selector：
选择前颜色_选择后颜色_有无置灰（true,false无置灰则不用谢）_圆角（corner无圆角不用谢）_linecorlor（边框颜色，与背景颜色一样不用写）_dashline（虚线则写）

drawable形状等：背景颜色（red）_圆角（corner）_线颜色（red）_虚线（无圆角则表示直角，无虚线则表示实线）

资源背景前置参数：

背景：bg_

RadioButton：rb_

CheckBox：cb_

ps：RadioButton，CheckBox 图片的话命名 rb_主题色_形状


其他：
通用style用通用方法命名，非通用activity主体名_控件名_标记属性

例如首页，下方radiobutton 命名方式：main_rb_bottom

颜色：color_未选择颜色_变换色_置灰色(没有则不写)

layout: 
Activity: ac_ 
fragment: fg_
Dialog: dialog_
Recyclerview： rv_
GridView: grid_


### android尺寸规范
#### 触摸目标大小
最小触摸目标大小48dp，图标间距24dp（详细请看下图），头像40dp

<img src="/img/jinjie/guifan/icon.png" />

网格式底部卡片间距 （我的标准是将下图24dp换成16dp,16dp换成12dp）

<img src="/img/jinjie/guifan/tab.png" />


#### 字体大小
```
    <dimen name="fourSize">8sp</dimen>
    <dimen name="fiveSize">10sp</dimen>
    <dimen name="sixSize">12sp</dimen>
    <dimen name="sevenSize">14sp</dimen>
    <dimen name="eightSize">16sp</dimen>
    <dimen name="nineSize">18sp</dimen>
    <dimen name="elevenSize">22sp</dimen>
   
 ```
    
#### 边距相关
```
    <dimen name="padding_half">4dp</dimen>
    <dimen name="padding">8dp</dimen>
    <dimen name="padding_doublehalf">12dp</dimen>
    <dimen name="padding_double">16dp</dimen>
    <dimen name="paddingX3">24dp</dimen>
    <dimen name="paddingX4">32dp</dimen>
    <dimen name="margin_half">6dp</dimen>
    <dimen name="margin">12dp</dimen>
    <dimen name="margin_double">24dp</dimen>
    <dimen name="marginX3">36dp</dimen>
    <dimen name="margin_normal">16dp</dimen>
    <dimen name="margin_normal_half">8dp</dimen>
    <dimen name="margin_normal_double">32dp</dimen>
```
标题栏：竖屏 48dp（有图标就要56dp，图标24dp，上下间距16dp）  横屏56dp  平板64dp

如下图：

<img src="/img/jinjie/guifan/dimen.png" />

#### 常用字体颜色

```
  <color name="gray">#ccc</color>
    <color name="lightgray">#eee</color>
    <color name="darkgray">#A9A9A9</color>
    <color name="black">#000</color>
    <color name="deepgray">#858689</color>
    <color name="lightblack">#505153</color>
    <color name="transblack">#333333</color>

```

<img src="/img/jinjie/guifan/color.png" />


#### 字体透明度

白色背景中，标准的文本透明度是87%。

次级文本/图标， 不透明度应该是48%。

提示 文本，处于还要更低的视觉层级，那么不透明度是24%

#### 字数

好的阅读体验，每行大约60个字符。恰当字符数量是提高可读性的关键

### 基本色设置

<img src="/img/jinjie/guifan/basecolor.png" />

其他关于UI性能优化可以参考：[android界面性能调优](https://centmeng.github.io/2016/09/21/android%E7%95%8C%E9%9D%A2%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98/)
