---
layout: post
title: "jquery validation plugin and select2"
date: 2013-03-29 11:18
comments: true
categories: ["技术"]
---

  [Jquery Validation Plugin](http://docs.jquery.com/Plugins/Validation)是非常棒的验证插件，select2也是一款很不错的select插件，让select更加强大与美观。

  Jquery Validation Plugin会默认忽视hidden的元素，而select2会把真正的select隐藏，而用自己的元素进行包裹，这时，select2就无法验证了。

  解决方案是：在初始化validator时，设置ignore参数

```ruby
    $form.validate
      rules:
        'user[channel]':
          required: true
      messages:
        'user[channel]':
          required: '请选择频道'
      #select2把真正的select隐藏了，而jquery validator会忽略需要特殊处理
      ignore: 'input[type=hidden]'
```
