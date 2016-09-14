---
title: TextView显示Gif图片实现图文混排
date: 2016-07-19 19:36:25
categories:
- Android技术
tags:
- 图文混排
---

<img src="/img/code.jpg" />

首先我们使用的html，解析成Spanned，然后设置Span来实现图文混排的，代码如下：

```

    public static Drawable getUrlDrawable(String source,  TextView mTextView) {
        GlideImageGetter imageGetter = new GlideImageGetter(mTextView.getContext(),mTextView);
        return imageGetter.getDrawable(source);

    }
       public static void setImageText(TextView tv){
       
       String  html ="<p dir=\"ltr\"><img src=\"http://p1.duyao001.com/image/article/a838e283f2b5d7cc45487c5fd79f84cb.gif\"><img src=\"http://statics.zhid58.com/Fqr9YXHd20fDOqil4nLAbBhNBw0A\"><br><br><img src=\"http://statics.zhid58.com/FufBg05KGCLypIvrYgjaXnTWySUS\"><br><br>OK咯木木木立刻哦lol额JOJO图谋女女look女女诺克各地测了测理论啃了了乐克乐克人咯咯JOJO图谋木木木木木木女女哦咯口头摸头LED可口女女LED咳咳JOJO咯JOJO咳咳咯科技JOJO扣女哦lol欧诺扣女<a href=\"http://www.taobao.com\">http://www.taobao.com</a> jvjvjvjv jgjvvjjvjce<br><br><img src=\"http://statics.zhid58.com/FkBcKMiLfzGfUpSb0bge4x-gIqWw\"><br></p><br>";
        Spanned htmlStr = Html.fromHtml(html);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            tv.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
            tv.setTextIsSelectable(true);
        }
        tv.setText(htmlStr);
        tv.setMovementMethod(LinkMovementMethod.getInstance());
        CharSequence text = tv.getText();
        if(text instanceof Spannable){
            int end = text.length();
            Spannable sp = (Spannable)tv.getText();
            URLSpan[] urls=sp.getSpans(0, end, URLSpan.class);
            ImageSpan[] imgs = sp.getSpans(0,end,ImageSpan.class);
            StyleSpan[] styleSpens = sp.getSpans(0,end,StyleSpan.class);
            ForegroundColorSpan[] colorSpans = sp.getSpans(0,end,ForegroundColorSpan.class);
            SpannableStringBuilder style=new SpannableStringBuilder(text);
            style.clearSpans();
            for(URLSpan url : urls){
                style.setSpan(url,sp.getSpanStart(url),sp.getSpanEnd(url),Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
                
                    ForegroundColorSpan colorSpan = new ForegroundColorSpan(Color.parseColor("#FF12ADFA"));
                    style.setSpan(colorSpan,sp.getSpanStart(url),sp.getSpanEnd(url),Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
                
            }
            for(ImageSpan url : imgs){
                ImageSpan span = new ImageSpan(getUrlDrawable(url.getSource(),tv),url.getSource());
                style.setSpan(span,sp.getSpanStart(url),sp.getSpanEnd(url),Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
            }
            for(StyleSpan styleSpan : styleSpens){
                style.setSpan(styleSpan,sp.getSpanStart(styleSpan),sp.getSpanEnd(styleSpan),Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
            }
            for(ForegroundColorSpan colorSpan : colorSpans){
                style.setSpan(colorSpan,sp.getSpanStart(colorSpan),sp.getSpanEnd(colorSpan),Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
            }

            tv.setText(style);
        }
    }
```

上面代码图片展示是通过ImageSpan来实现的，但默认的图片展示的gif图片是静态取第一帧图片，我们可以在获取图片时候使用[Glide](http)，来实现播放gif,glide是图片加载库，这个库被广泛的运用在google的开源项目中，包括2014年google I/O大会上发布的官方app。Glide和Picasso有90%的相似度，准确的说，就是Picasso的克隆版本。但是在细节上还是有不少区别的。而且性能上更加优化。

把Glide引入到我们项目中，然后在创建UrlDrawable 和 GlideImageGetter

代码可以参考：[RichTextView](https://github.com/CentMeng/RichTextView)

方法调用就是

```
  ImageTextUtil.setImageText(tv,html);
```


需注意的是
    //个别机型如魅族手机，如果使用以下语句会报错导致无法显示Textview内容 View too large to fit into drawing cache 的Warning
//        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
//            tv.setLayerType(View.LAYER_TYPE_SOFTWARE, null); 关闭硬件加速
//            tv.setTextIsSelectable(true); 设置是否可以粘贴
//        }