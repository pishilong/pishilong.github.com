---
layout: post
title: "ckeditor的逆袭"
date: 2012-06-28 11:39
comments: true
categories: ['技术']
keywords: ckeditor, rails
---
原本用kindeditor毫无压力，也算支持国产，但是无奈，为了整合数学公式编辑器，只能改用ckeditor或者tinyMCE这两个主流大哥。

而tinyMCE又与Rails不能很好集成，只有原生的插件。一没有gem，二不能上传文件。

于是改用比较成熟的[ckeditor](https://github.com/galetahub/ckeditor)，整合rails相当方便，上传等也一应俱全,还提供ActiveRecord, Mongid等持久化支持，和[paperclip](https://github.com/thoughtbot/paperclip),[carrierwave](https://github.com/jnicklas/carrierwave)上传插件的支持。

    gem "ckeditor", "3.7.1"
    rails generate ckeditor:install --orm=active_record --backend=paperclip

一直使用很顺畅，直到碰到两个很恶心的问题:

* ckeditor总会把内容自己套一层<p>标签

  这是因为ckeditor对于换行有三种模式，默认是<p>,改成br模式或者div模式即可解决问题，在config.js中加入:
{% highlight ruby %}
config.enterMode = CKEDITOR.ENTER_BR;
config.shiftEnterMode = CKEDITOR.ENTER_BR;
{% endhighlight %}

* HTML里面本来<table>标签在<span>标签里面，在kindeditor里面，能正常显示在富文本编辑器里，但是ckeditor会自己把外面的<span>标签移除，然后内嵌到table里的文字周围。

  原本以为是ckeditor自己怪异的行为，然后几乎花了一天时间去看ckeditor的源码以及详细设置，使用了很多方案:
{% highlight ruby %}
config.removePlugins = 'scayt';
config.autoParagraph = false;
config.ignoreEmptyParagraph = false;
{% endhighlight %}

  都不行，后来在一个帖子里看到，span标签属于inline element,而table属于block element,从规范和合理的角度出发,block是不能内嵌到inline element的。
  所以ckeditor进行了检查并处理。确实在ckeditor的dtd中也对各类标签进行了定义

ckeditor总的来说还是很不错，但是体积比Kindeditor稍大，如果不是特别的需求，我会选择kindeditor.

下一篇着重讲下数学公式编辑器的集成，从官方的php移植到了rails上。

