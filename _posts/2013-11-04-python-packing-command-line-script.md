---
layout: post
title: "python命令行程序打包"
description: "打包python程序，并通过命令行调用"
category: "python"
tags: [setuptools, python]
---
{% include JB/setup %}
我们知道在linux下通过命令行调用程序其实就是先把程序编译生成二进制文件，然后再把文件放入到PATH中，就可以直接通过文件名进行调用了，这里python程序也不例外。前面讲过了如何将一个python程序打包并发布，这里就讲一下命令行的python打包发布和下载安装过程。

我们还是利用前面博客中的myproject为例，在原来的基础上添加一些代码：
{% highlight sh %}
.
├── setup.py
└── src
    ├── bin
    │   ├── command-line
    │   └── myproject
    └── myproject
        ├── command_line.py
        ├── __init__.py
        └── __init__.pyc

3 directories, 6 files
{% endhighlight %}
