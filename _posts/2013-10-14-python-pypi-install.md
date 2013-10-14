---
layout: post
title: "python pypi 私有源搭建"
description: ""
category: ""
tags: []
---
{% include JB/setup %}
今天学习了pypi的私有源的搭建。过程很曲折，出现了很多问题，这里做个记录，希望以后的人能有所借鉴，自己也做一下总结……

##什么是pypi
这个问题可以从[官方文档](https://pypi.python.org/pypi)得到答案。pypi的全称是：The Python Package Index，翻译过来就是一个python包的索引，python包存储在一个统一管理的仓库中，然后为所有在这个仓库的软件生成一个列表,这就是pypi.
##如何使用pypi
用过python的同学都知道，通过下面三个命令都可以安装python包。但是它们之间的区别又是什么呢？
{% highlight sh %}
    1. python setup.py  install
{% endhighlight %}
{% highlight sh %}
    2. pip install packagename
{% endhighlight %}
{% highlight sh %}
    3. easy_install packagename
{% endhighlight %}
第一种方式就是先把要安装的包的模块下载到本地，然后进入解压目录，执行上面的命令。
第二种和第三种都是直接输入命令，就会自动从pipy库中下载相应的包然后安装的预先定义好的路径。
如果不是要修改源码，第二种和第三种方式无疑是最好的
###为什么要搭建pypi私有源
我们知道pypi就是一个存储软件的仓库，
