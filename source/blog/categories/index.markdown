---
layout: page
title: "Categories"
date: 2012-10-10 01:07
comments: false
sharing: true
footer: true
---

<ul>
{% for item in site.categories %}
    <li><a href="/blog/categories/{{ item[0] }}/">{{ item[0] | capitalize }}</a> 
	(<a href="/blog/categories/{{ item[0] }}/atom.xml">RSS</a>) [{{ item[1].size }}]</li>
{% endfor %}
</ul>
