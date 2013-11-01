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
第8行表示这个工程的名称，第9行是这个工程的版本号，第10行表示对指定目录的文件打包，其他信息请查看相关文档，这里先对用到的配置项进行讲解  
**3, 通过命令行生成工程文件**  
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
我们可以看到在dist文件夹下生成了一个egg文件，通过使用命令file可以查看这个文件的格式：
{% highlight sh %}
    $ file dist/myproject-0.1.0-py2.6.egg
    dist/myproject-0.1.0-py2.6.egg: Zip archive data, at least v2.0 to extract
{% endhighlight %}
其实egg文件就一个压缩包,关于egg的介绍请参考[这篇博客](http://peak.telecommunity.com/DevCenter/PythonEggs)  
**4, 写自己的程序**  
在当前目录下建立一个文件夹，名字与setup.py里写的name字段保持一致,然后在进入目录，创建__init__.py文件
{% highlight sh%}
    $ mkdir myproject
    $ cd myproject
    $ touch __init__.py
{% endhighlight %}
文件\_\_init\_\_.py内容如下：
{% highlight python %}
  1 #!/usr/bin/env python
  2 #-*-coding:utf-8-*-
  3 
  4 def test():
  5     print "Hello World!"
  6 if __name__ == '__main__':
  7     test()
{% endhighlight %}
再次运行生成egg的命令：

{% highlight sh %}
   python setup.py bdist_egg 
{% endhighlight %}
这时整个目录结构变成了下面的情况：
{% highlight sh %}
.
├── build
│   ├── bdist.linux-x86_64
│   └── lib
│       └── myproject
│           └── __init__.py
├── dist
│   └── myproject-0.1.0-py2.6.egg
├── myproject
│   └── __init__.py
├── myproject.egg-info
│   ├── dependency_links.txt
│   ├── not-zip-safe
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   └── top_level.txt
└── setup.py
{% endhighlight %}
可以看一下这个egg文件的内容：
{% highlight sh %}
    $ unzip -l dist/myproject-0.1.0-py2.6.egg 
      Archive:  dist/myproject-0.1.0-py2.6.egg
        Length      Date    Time    Name
      ---------  ---------- -----   ----
            118  11-01-2013 14:35   myproject/__init__.py
            354  11-01-2013 14:35   myproject/__init__.pyc
            232  11-01-2013 14:35   EGG-INFO/PKG-INFO
            194  11-01-2013 14:35   EGG-INFO/SOURCES.txt
              1  11-01-2013 14:35   EGG-INFO/dependency_links.txt
              1  10-21-2013 12:29   EGG-INFO/not-zip-safe
             10  11-01-2013 14:35   EGG-INFO/top_level.txt
      ---------                     -------
            910                     7 files
{% endhighlight %}
可以看到里面多出来一个myproject文件夹和它所包含的文件，正是我们刚才创建的。其实到这里我们的一个简单的python包已经完成了，我们可以安装使用了,方法如下：  
**5, 源代码的安装和使用**  

{% highlight sh %}
$ sudo python setup.py install
  running install
  running bdist_egg
  running egg_info
  ... 
  ... 
  Installed /root/.local/lib/python2.6/site-packages/myproject-0.1.0-py2.6.egg
  Processing dependencies for myproject==0.1.0
  Finished processing dependencies for myproject==0.1.0
$ python -c "from myproject import test; test()"
  Hello World!
{% endhighlight %}
可以看到我们的包已经被安装了，并且可以运行我们之前在包里封装的方法。  
**6, 代码结构规划**  
一般情况下为了使代码看起来更加有条理，我们把源代码放到src文件夹下。首先把myproject文件夹移动到src中，然后删除以前产生的文件，只保留自己写的问文件，其结构大致如下：

{% highlight sh %}
   ├── setup.py
└── src
    └── myproject
        ├── __init__.py
        └── __init__.pyc 
{% endhighlight %}
setup.py内容也要修改，修改默认的搜索路径为src，而不是从根目录开始，内容如下：
{% highlight python %}
#setup.py
#!/usr/bin/env python
#-*- coding:utf-8 -*-

from setuptools import setup, find_packages

setup(
        name = "myproject",        
        version="0.1.0", 
        packages = find_packages('src'), 
        package_dir = {'':'src'},
        zip_safe = False,

        description = "egg test demo.", 
        long_description = "egg test demo, haha.", 
        author = "sean",
        author_email = "seanerchen@gmail.com",

        license = "GPL",
        keywords = ("test", "egg"),
        platforms = "Independant",
        url = "",
        )
{% endhighlight %}
然后运行命令生成egg等文件：
{% highlight sh %}
$ python setup.py bdist_egg

$ tree
.
├── build
│   ├── bdist.linux-x86_64
│   └── lib
│       └── myproject
│           └── __init__.py
├── dist
│   └── myproject-0.1.0-py2.6.egg
├── setup.py
└── src
    ├── myproject
    │   ├── __init__.py
    │   └── __init__.pyc
    └── myproject.egg-info
        ├── dependency_links.txt
        ├── not-zip-safe
        ├── PKG-INFO
        ├── SOURCES.txt
        └── top_level.txt

8 directories, 10 files
{% endhighlight %}
然后用同样的方式进行安装运行，依然没有问题。  
**7, 卸载程序**  
安装过的python包，如果要删除该如何做呢？
首先要找到easy-install.pth文件,这个文件是自动查询安装包的路径的文件，如果把安装包的路径从这个文件中去掉，将找不到要使用的包
{% highlight sh %}
$ locate easy-install.path
  /root/.local/lib/python2.6/site-packages/easy-install.pth
  /search/virtual/pypienv/lib/python2.6/site-packages/easy-install.pth
  /usr/lib/python2.6/site-packages/easy-install.pth
  /usr/lib64/python2.6/site-packages/easy-install.pth
{% endhighlight %}
亲测第一个文件是我们要找的文件，具体原因现在还没研究，先能用再说吧。  
{% highlight sh%}
#ps:当运行安装程序发现最后几行信息:
...
Processing myproject-0.1.0-py2.6.egg
creating /root/.local/lib/python2.6/site-packages/myproject-0.1.0-py2.6.egg
Extracting myproject-0.1.0-py2.6.egg to /root/.local/lib/python2.6/site-packages
Adding myproject 0.1.0 to easy-install.pth file

Installed /root/.local/lib/python2.6/site-packages/myproject-0.1.0-py2.6.egg
Processing dependencies for myproject==0.1.0
Finished processing dependencies for myproject==0.1.0
#*可以看出包的安装路径和包的路径信息都能找到，删除相应的包和路径就会发现无法使用该包了。
{% endhighlight %}
打开文件，删除./myproject-0.1.0-py2.6.egg这一行，再次运行python时就会提示找不到该包
{% highlight sh %}
>>> from myproject import test
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named myproject
{% endhighlight %}
**8, 打包上传和安装**  
这一步和前面的私有源建立的过程一样，不再赘述了，详情请参考**python pypi 私有源搭建**这篇博客
