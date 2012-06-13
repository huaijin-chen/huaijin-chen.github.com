---
layout: page
title: Huaijin
tagline: Huaijin-Chen
---
{% include JB/setup %}

Read [Jekyll Quick Start] (http://jekyllbootstrap.com/usage/jekyll-quick-start.html) huaiaiem

#Aboute Me

##Basic
My name is ChenChao(陈超).I always use `huaijin` in the Internet .
		
Follow me in sina: [@huaijin](http://www.weibo.com/huaijin)

If you can thougth out GFW,you can 
		
Follow me in twitte: [@huaijin](http://www.twitter.com/huaijin) 

If you have some questions or problems,contact me by Email or WEiBO
	
My Email: (If Unixer/Linuxer you are)

**`echo zherorjkccisverynb.v | tr ebszroyvj.hckin acghijlmnou15@.`**

##Intrestings
1. I love reading books,
	for example, literure,Math,Biology,psy and so on.
2. I am a guy in Computer Science,
	my resereach in Computer Vision & Machine Learning.
3. I am also a travler.


	


#Blog List

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

