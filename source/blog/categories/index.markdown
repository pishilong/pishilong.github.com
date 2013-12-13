---
layout: page
title: "Categories"
date: 2013-12-13 11:13
comments: true
sharing: true
footer: true
---
# 分类

<ul>
{% for item in site.categories %}
    <li><a href="/blog/categories/{{ item[0] }}/">{{ item[0] | capitalize }}</a> [ {{ item[1].size }} ]</li>
{% endfor %}
</ul>
