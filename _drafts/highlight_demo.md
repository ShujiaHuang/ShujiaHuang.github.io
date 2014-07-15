---
layout: post
title: 'Demo'
---

{% highlight html%}
<div id="back-top">
  <a href="#top" title="回到顶部"></a>
</div>
{% endhighlight %}

## 二、添加js代码

使用 js 响应窗口滚动事件和按钮的点击事件，其中 100 表示向下滚动 100 个像素时出现按钮，800 表示使用 800ms 的时间滚动到顶部。

{% highlight javascript %}
$("#back-top").hide();
$(document).ready(function () {
  $(window).scroll(function () {
    if ($(this).scrollTop() > 100) {
      $('#back-top').fadeIn();
    } else {
      $('#back-top').fadeOut();
    }
  });
  $('#back-top a').click(function () {
    $('body,html').animate({
      scrollTop: 0
    }, 800);
    return false;
  });
});
{% endhighlight %}

## 三、添加样式

此处使用一个背景透明的向上箭头图片，通过 `backgroud-color` 指定正常时和鼠标放上时的背景颜色，`border-radius` 指定圆角的半径。

{% highlight javascript %}
#back-top {
  position: fixed;
  bottom: 30px;
  margin-left: 1040px;
}
#back-top a {
  width: 54px;
  height: 54px;
  display: block;
  background: #ddd url(../img/bgs/bg_up_arrow.png) no-repeat center center;
  background-color: #aaa;
  -webkit-border-radius: 7px;
  -moz-border-radius: 7px;
  border-radius: 7px;
  -webkit-transition: 1s;
  -moz-transition: 1s;
  transition: 1s;
}
#back-top a:hover {
  background-color: #777;
}
{% endhighlight %}
