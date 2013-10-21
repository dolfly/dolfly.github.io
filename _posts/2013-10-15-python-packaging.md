---
layout: post
title: "python打包"
description: ""
category: "python"
tags: [python, setuptools]
---
{% include JB/setup %}
python的打包方式有很多种，而且针对不同的平台又有很多不同的方式，这里指针对linux平台进行介绍……

##1 python打包工具选择  
关于python打包相关的工具，这篇文档有一个[列表](https://python-packaging-user-guide.readthedocs.org/en/latest/projects.html#)，其中打包工具主要分为pip, dislib, setuptools, distribute(已经于setuptools合并）。还有博客也是关于python 打包工具的，[这篇博客](http://www.ituring.com.cn/article/19090)对于python的各种打包工具，及其优缺点写的很详细，里面说的Distutils2是更为先进的打包工具，但是适用范围还不是太广泛，这里先不做研究了。经过综合对比，目前选中setuptools和pip这两个个比较成熟的打包工具，下面就对他们用法和原理分别进行详细的介绍。  

###2 setuptools
[setuptools的官方文档地址](http://peak.telecommunity.com/DevCenter/setuptools)  
####2.1 使用方法
首先来一个简单的上手过程，通过这个过程可以快速了解setuptools怎么用的，详情请移步博客[python egg 学习笔记](http://www.worldhello.net/2010/12/08/2178.html)
看完这个教程就能自己做一个简单的包了，但是这里只是涉及了一种包，就是python import 引过来，然后调用其中的函数，还有一种也很普遍的方式就是命令行方式，例如jekyll命令，sentry命令等，这种方式其实可以参考[这篇文档](http://www.scotttorborg.com/python-packaging/index.html)。跟随这两篇博客基本就可以满足我们的需求了
这里对其安装步骤进行一下总结：  
**1, 安装setuptools**
{% highlight sh%}
    $ wget http://peak.telecommunity.com/dist/ez_setup.py
    $ sudo python ez_setup.py
{% endhighlight %}
其实安装的方式有很多种，具体方法参见前面提到的博客  
**2, 建立一个工程**
{% highlight sh%}
    $ mkdir mypackage 
    $ cd mypackage
    $ touch setup.py
{% endhighlight %}
setup.py的内容如下：
{% highlight sh%}
  1 #setup.py
  2 #!/usr/bin/env python
  3 #-*- coding:utf-8 -*-
  4 
  5 from setuptools import setup, find_packages
  6 
  7 setup(
  8         name = "myproject",
  9         version="0.1.0",
 10         packages = find_packages(),
 11         zip_safe = False,
 12 
 13         description = "egg test demo.",
 14         long_description = "egg test demo, haha.",
 15         author = "sean",
 16         author_email = "seanerchen@gmail.com",
 17 
 18         license = "GPL",
 19         keywords = ("test", "egg"),
 20         platforms = "Independant",
 21         url = "",
 22         )
{% endhighlight %}
第8行表示这个工程的名称，第9行是这个工程的版本号，第10号表示对指定目录的文件打包，其他信息请查看相关文档，这里先对用到的配置项进行讲解  
这是一个空的工程就建好了，我们可以通过命令生成工程所需要的其它目录和文件：
{% highlight sh %}
   python setup.py bdist_egg 
{% endhighlight %}
命令执行后，myproject目录的的结构如下：
{% highlight sh %}
    .
    ├── build
    │   └── bdist.linux-x86_64
    ├── dist
    │   └── myproject-0.1.0-py2.6.egg
    ├── myproject.egg-info
    │   ├── dependency_links.txt
    │   ├── not-zip-safe
    │   ├── PKG-INFO
    │   ├── SOURCES.txt
    │   └── top_level.txt
    └── setup.py
{% endhighlight %}

###3. pip
