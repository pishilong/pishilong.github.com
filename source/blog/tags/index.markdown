---
layout: page
title: "tags"
date: 2013-12-13 13:01
comments: false
sharing: false
footer: false
---
# 标签

<ul>
{% for item in site.tags %}
    <li><a href="/blog/tags/{{ item[0] }}/">{{ item[0] | capitalize }}</a> [ {{ item[1].size }} ]</li>
{% endfor %}
</ul>
