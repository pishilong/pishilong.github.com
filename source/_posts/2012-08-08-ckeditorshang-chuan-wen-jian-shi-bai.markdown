---
layout: post
title: "ckeditor上传文件失败"
date: 2012-08-08 11:17
comments: true
categories: ['技术']
---
Rails: 3.2.6

使用的是Ckeditor的Rails Gem.

突然不能上传文件，控制台显示在上传插入数据库时直接rollback了

于是重载app/controllers/ckeditor/application_controller.rb的respond_with_asset(asset)方法，

将asset.save,改为asset.save!

查看抛出的异常,是文件大小超出范围,显示必须在0-2048 Bytes之间，我还以为是文件没有拷贝过去，于是大小是0。

这时辉神出现了，他指出，2.megabytes不应该是2048啊。于是测试ActiveSupport，这个没问题，网上一搜，才发现，god这个gem重载了这个方法.于是2.megabytes变成了2048

但是最终没有把god注销，而是在代码中直接使用了2*2048*2048

