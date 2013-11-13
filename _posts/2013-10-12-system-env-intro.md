---
layout: post
title: "软件开发环境配置规范"
description: ""
category: ""
tags: []
---
{% include JB/setup %}
# 软件开发环境配置规范

## 一.规范的意义
没有规矩，不成方圆。 这话应该都不陌生，在服务器使用方面，如果没有一个规范进行指导和约束，管理员和开发人员可以随意配置使用服务器，会造成了很多问题：

1.系统维护管理的难度和成本越来越大

2.系统部在完成服务器的基本安装后，交由应用部门使用的之后将可能完全不受控制

3.服务器配置无法统一，开发环境配置无法统一

4.开发人员随意配置服务器开发环境，造成安全隐患，增加系统维护管理的难度和成本

5.开发环境不统一，程序开发和移植、安装等工作会增加不必要的麻烦，同时也增加了开发人员熟悉服务器环境甚至重新配置服务器的麻烦

因此希望讨论、制定并逐步完善一个比较通用的适应大多数开发需求的开发环境配置规范，由系统部在完成系统基本安装后，按照规范完成开发环境的安装和配置，达到交给应用部门的服务器是可直接使用、不必重新配置的。

由于是初稿，规范未必合理，但是有规范总比没有规范好，可以经过实践和讨论不断完善规范。

## 二.系统环境说明

### 1软件开发的4个环境
1.内部开发环境

项目初期开发使用，开发人员可以登陆服务器，并且可以看到服务器的调试错误信息。

一般部署于公司内部机房，使用和外部生产环境相同配置的软件（Linux/Windows,Apache/Nginx,PHP/Python/Perl,MySQL,Memcache,Redis等）环境（采用cfengine来管理、同步这些服务器的配置信息）。

其他配置也基本一样，使用独立的数据库和存储（无法访问生产环境的数据库和存储服务器）以及四层交换环境。。

2.内部测试环境

项目开发过程中用于测试的环境，配置和内部开发环境相同。

3.生产环境测试

项目上线运营之后，用于线上故障诊断，性能分析等用途的测试服务器，开发人员可以登陆服务器，并且可以查看程序的调试错误信息。
使用生产环境的数据库和存储系统，不提供对外的访问。

4.生产环境运营

项目运营，程序上线后的正式运行环境，不提供开发人员访问，不打开错误调试信息，会有其他比较严格的访问控制。

### 2开发环境中的各类服务器说明

1.WEB前端服务器集群

主要安装Apache,Nginx,PHP,Python,Memcache,Redis等软件。 通过四层交换机或HA做负载均衡。

2.MySQL数据库集群

主要安装MySQL数据库，分主库（读写库）和从库（只读库）两类，每个数据库服务器会运行多个数据库副本，分别监听不同的端口，每个数据库副本提供一个或者多个项目使用。

主库会有备份库，以解决主库宕机后从库自动投入服务。
从库有多台服务器，使用四层交换机或HA实现读操作的负载均衡。

3.存储服务器集群

使用多台服务器提供NFS网络文件服务。

4.测试服务器

测试服务器配置和Web前端相同

5.程序后端服务器

提供代码发布服务，开放rsync服务。

6.管理后台服务器

各个项目投入使用后，会提供运营部门管理运营，供他们使用的管理后台程序统一安装在该类服务器上，项目申请时，会开放从该服务器访问Rsync,MySQL等服务的授权。
该服务器只开放https协议以保证数据传输过程的保密性。

## 三.规范

### 目录布局和文件命名规范

#### 采用方法

目录命名规范主要是为了统一维护方便，也为了方便使用，可以通过下面的几个方法达到统一目录布局和查找的目的：

1.mount 直接创建相应的 mount 加载点如 /sinasrv/www

```
mount --bind 参数，通过这个方法可以把一个目录挂载到另一个目录下，例如：
mount --bind /sinasrv/www/htdocs /data1/apache/htdocs mount --bind /sinasrv/www/logs /data2/apache/logs
mount --bind /sinasrv/www/mmcache /data2/mmcache
```

2.符号链接 (这个最常用)

#### 常用目录布局规范

明确划分目录的好处是我们可以给不同类型使用的目录配置不同的访问权限，以此来达到较高的安全控制。

sinasrv为项目组标识

:        |目录          |权限       |用途
---------|--------------------|-------------------------------------------------
缓存目录  |cache、*cache |读写      |用来保存有一定生存期限的临时数据，不提供数据备份服务。只供本地访问的目录cache；使用nfs文件系统共享的目录ncache；
数据目录  |data、*data   |读写      |用来保存具有永久存在期限的数据，提供备份服务，默认每日自动做一次备份。只供本地访问的数据目录data;使用nfs文件系统共享的目录ndata
私有目录 |privdata       |读     |用来保存不希望从web访问的数据，如配置信息，提供备份。只供本地访问的数据目录privdata;
日志目录 |logs、*logs     |读写     |用来提供记录日志。应用程序日志记录applogs;
上传目录 |upload、uploads、userupload、useruploads|读|用来提供上传数据存储
文档目录 |www           |读   |web服务存储代码,读权限

本地的 cache, data,uploads 目录将会配置同名的 alias 提供用户从 Web 访问.


###### 软件包安装时的目录布局

该规范适用于大多的基础开发库软件和一些工具软件
例如libiconv,libpng,zlib
例如stow,php,perl
软件根目录和子目录布局

```
--prefix = /usr/local/sinasrv
--exec_prefix = %{_prefix}
--bindir = %{_exec_prefix}/bin
--sbindir = %{_exec_prefix}/sbin
--libexecdir = %{_exec_prefix}/libexec
--datadir = %{_prefix}/share
--sysconfdir = %{_prefix}/etc
--sharedstatedir= %{_prefix}/com
--localstatedir = %{_prefix}/var
--lib = lib
--libdir = %{_exec_prefix}/%{_lib}
--includedir = %{_prefix}/include
--oldincludedir = /usr/include
--infodir = %{_prefix}/info
--mandir = %{_prefix}/man
```

##### 程序安装目录、数据根目录命名

：-             |目录/示例                                       |说明/规则
----------------|-----------------------------------------------|----------------------------
软件安装根目录    |/usr/local/sinasrv                             |
Apache安装目录   |/usr/local/sinasrv/apache                      |
MySQL安装目录    |/usr/local/sinasrv/mysql                       |
数据根目录命名    |/sinasrv                                       |

##### 本地磁盘使用和目录命名

###### 磁盘目录
假设都为3块硬盘的服务器，系统盘不用来保存数据，第一块数据库盘加载在/data1
目录下，第二块数据盘加载在/data2目录下。

/data1 磁盘创建如下目录:

```
/data1/sinasrv/www/htdocs
/data1/sinasrv/www/data
/data1/sinasrv/www/cgi-bin
/data1/sinasrv/mysql/
```

/data2 磁盘创建如下目录:

```
/data2/sinasrv/www/logs
/data2/sinasrv/www/cache
/data2/sinasrv/www/mmcache
/data2/sinasrv/www/phpsession
/data2/sinasrv/www/userupload
/data2/sinasrv/mysql/
```

对于1块数据盘的服务器，将数据盘加载在 /data1 目录下，所有的目录都创建在
/data1/sinasrv 目录下。

对于3块数据盘的服务器，将第三块数据盘加载在 /data3 目录下，创建 /data3/sinasrv 目录，以后根据需要再使用。

…

上面的各个目录通过符号链接到 /sinasrv/www 相应目录下，如果在 chroot 环境下，则通过 mount --bind 加载到 /sinasrv/www 目录下。

###### 实现兼容性的目录命名
通过符号链接或者 mount --bind 方法实现一些习惯使用的固定目录。

符号链接 /usr/local/apache 到 /usr/local/sinasrv/apache

符号链接 /data1/apache/htdocs 到 /sinasrv/www/htdocs

符号链接 /usr/local/mysql 到 /usr/local/sinasrv/mysql

符号链接 /data2/mysql 到 /usr/local/sinasrv/mysql



##### Apache 

###### 相关目录、文件路径的命名
####### 虚拟主机的文档、程序、数据的目录命名

：-             |目录/示例                                       |说明/规则
----------------|-----------------------------------------------|----------------------------
Apache文档根目录 |/sinasrv/www                                   |
Apache文件目录   |/sinasrv/www/htdocs/survey.news.sina.com.cn    |根目录 + 文档目录 + 虚拟主机域名
Apache cgi程序的目录|/sinasrv/www/cgi-bin/survey.news.sina.com.cn |根目录 + 文档目录 + 虚拟主机域名
Apache 程序数据目录|/sinasrv/www/data/survey.news.sina.com.cn     |根目录 + 数据目录 + 虚拟主机域名
Apache 程序缓存目录|/sinasrv/www/cache/survey.news.sina.com.cn    |根目录 + 缓存目录 + 虚拟主机域名 
Apache 基于 nfs 的共享数据目录|/sinasrv/www/ncache/survey.news.sina.com.cn|根目录 + 缓存目录 + 虚拟主机域名
Apache 基于 nfs 的共享文档目录|/sinasrv/www/nhtdocs/survey.news.sina.com.cn|
Turck MMCache 的程序缓存目录|/sinasrv/www/mmcache      |
PHP Session 的目录 |/sinasrv/www/phpsession           |
用户文件上传目录    |/sinasrv/www/userupload            |

说明：上面关于虚拟主机的配置可以看到，虚拟主机使用的各个特定需求的目录都是定义在每个特定目录下的，比如data，是所有虚拟主机的data目录的根，每个虚拟机主机已域名创建自己的目录。这样做的好处是可以统一控制data的权限，而不用把data目录创建在每个虚拟机主机的根目录下。

####### 虚拟主机的日志目录和日志文件命名
：-             |目录/示例                                       |说明/规则
----------------|-----------------------------------------------|----------------------------
日志根目录        |/sinasrv/www/logs                               |
错误日志          |/sinasrv/www/logs/error/survey.news.sina.com.cn-error_log|日志根目录 + 错误日志目录 + 虚拟主机域名 + "-error_log"后缀
访问日志          |/sinasrv/www/logs/access/survey.news.sina.com.cn-access_log|日志根目录 + 错误日志目录 + 虚拟主机域名 + "-access_log"后缀
归档日志          |/sinasrv/www/logs/archive/051231/survey.news.sina.com.cn_10.44.6.24_cn.gz|日志根目录 + 归档日志目录 + 日期目录 + 虚拟主机域名 + 外网IP地址 + "_cn"+后缀 

access_log 文件默认不记录图片访问记录。
Apache access_log 每日临晨0点自动归档，每日生成一个日志文件，按日期保存在日志归档目录下，自动压缩成 gzip 格式。
Apache error_log 日志文件不归档，每日临晨0点自动清空。
默认访问日志只保存在本地服务器，存留期为30天，超出后自动删除。

###### 编译规范

各个模块都采用 DSO 方式编译，启动时只选择常用的模块，如下:

```
   http_core.c
   mod_log_config.c
   mod_mime.c
   mod_include.c
   mod_dir.c
   mod_cgi.c
   mod_alias.c
   mod_rewrite.c
   mod_access.c
   mod_auth.c
   mod_so.c
   mod_env.c
   mod_negotiation.c
   mod_setenvif.c
   mod_php5.c 
```  

###### 默认配置

httpd.conf 配置文件

基本配置如下：

```
   ServerType standalone
   ServerRoot "/usr/local/sinasrv/apache"
   PidFile /sinasrv/www/logs/httpd.pid
   ScoreBoardFile /sinasrv/www/logs/httpd.scoreboard
   Timeout 120
   KeepAlive On
   MaxKeepAliveRequests 10240
   KeepAliveTimeout 10
   MinSpareServers 15
   MaxSpareServers 30
   StartServers 15
   MaxClients 1024
   MaxRequestsPerChild 0
   Port 80
   User www
   Group www
   ServerAdmin tongjian@staff.sina.com.cn
   ServerName 127.0.0.1
   DocumentRoot "/sinasrv/www/htdocs/default"
   <Directory />
    Options FollowSymLinks
    AllowOverride None
   </Directory>
   LogFormat "%h %l %u %t %T \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
   CustomLog /sinasrv/www/logs/access_log combined
   ServerSignature Off
   ServerTokens ProductOnly
   DirectoryIndex index.shtml index.html index.php
   AccessFileName .htaccess
   <Files ~ "^\.ht">
    Order allow,deny
    Deny from all
    Satisfy All
   </Files>
   UseCanonicalName On
   HostnameLookups Off

   <IfModule mod_setenvif.c>
       SetEnvIfNoCase Request_URI "\.(gif|js|jpg|swf)$" dontlog
   </IfModule>

   <FilesMatch "\.(sh|log|txt|sql|bak|bak2|old|1|2|php~)$">
               Deny from all
   </FilesMatch>
   
   ErrorDocument 404 "Hi, The requested URL was not found on this server !"
   ErrorDocument 500 "Ooooooooooooops, I don't know what's happen !"
   
   NameVirtualHost *:80
```

虚拟主机配置格式如下

```
   <VirtualHost *:80>
       ServerName news.survey.sina.com.cn
       ServerAdmin minghui@staff.sina.com.cn
       DocumentRoot /sinasrv/www/htdocs/news.survey.sina.com.cn/
       ErrorLog /sinasrv/www/htdocs/error/news.survey.sina.com.cn-error_log
       CustomLog /sinasrv/www/htdocs/error/news.survey.sina.com.cn-access_log combined env=!dontlog
       Alias "/data/"  "/sinasrv/www/data/news.survey.sina.com.cn/"
       Alias "/cache/" "/sinasrv/www/cache/news.survey.sina.com.cn/"
       SetEnv SINASRV_DATA_DIR "/sinasrv/www/data/news.survey.sina.com.cn/"
       SetEnv SINASRV_CACHE_DIR "/sinasrv/www/cache/news.survey.sina.com.cn/"
       SetEnv SINASRV_NFSDATA_DIR "/sinasrv/www/nfsdata/news.survey.sina.com.cn/"
       SetEnv SINASRV_NFSCACHE_DIR "/sinasrv/www/nfscache/news.survey.sina.com.cn/"
       SetEnv SINASRV_NFSHTDOCS_DIR "/sinasrv/www/nfshtdocs/news.survey.sina.com.cn/"
   </VirtualHost>
```
##### PHP
###### 相关目录、文件路径的命名
###### 编译规范

采用DSO模式编译为 Apache 动态加载模块，需要先安装 Apache.
扩展模块以DSO方式编译安装，启动时只加载需要使用的模块：

```
   ctype
   gd
   iconv
   mysql
   overload
   pcre
   posix
   session
   sina
   standard
   tokenizer
   Turck MMCache
   xml
```
###### 默认配置
php 使用 php.ini-recommended 作为模板配置文件，在此基础之上增加一些安全
方面的设置。
下面是对部分配置的说明

```
   - output_buffering = 4096   性能调节参数，默认是关闭的。
   - variables_order = "GPC"  性能调节参数，去掉了不常用的环境变量，
     只用Get,POST,Cookie，如果访问环境变量可以考虑调用getenv()
     目前的配置是 "GPCS"，不知道可不可以去掉 Server环境变量？
   - error_reporting  =  E_ALL 安全因素
   - allow_call_time_pass_reference = Off 代码清洁
   - expose_php = Off 安全因素
   - register_argc_argv = off 命令行参数，在CLI SAPI方式中才有用
   - magic_quotes_gpc = off 关闭系统自动escape用户输入，
     用户可以根据需要使用下列函数自己进行escape：
     addslashes(), escapeshellcmd(), mysql_escape_string()
   - allow_persistent = On
     目前全部开放，考虑到个别程序的错误会引起MySQL的不稳定，以后可能会给个
     别vhost开放这个特性，也可能会在mysql数据库中限制最大连接数或者设置
     timeout，或者设置php中的关于database的max_links and max_persistent
   - safe_mode, open_basedir and memory_limit
     处于性能考虑，不推荐使用上面的指令，the checks performed by 
     PHP to enforce them are quite expensive and can lead
     to significant performance losses if enabled.   
   - display_errors Off 生产环境关闭错误显示

php.ini安全配置

默认打开安全模式
disable_functions = phpinfo, exec, system, passthru, shell_exec
禁用一些不安全的函数
open_basedir = /sinasrv/www/htdocs/survey.news.sina.com.cn:/sinasrv/www/cache/survey.news.sina.com.cn:/sinasrv/www/data/survey.news.sina.com.cn:/sinasrv/www/nfscache/survey.news.sina.com.cn:/sinasrv/www/nfsdata/survey.news.sina.com.cn
限定程序只能访问这个目录，该功能不受 safe_mode 关闭/打开 的影响

上面的指令也可能会在 httpd.conf 配置文件中实现
<Directory /docroot>
php_admin_value open_basedir /sinasrv/www/htdocs/survey.news.sina.com.cn; /sinasrv/www/cache/survey.news.sina.com.cn
</Directory>
安全模式下可以包含的库路径
safe_mode_include_dir = ".:/usr/local/sinasrv/lib/php/"
安全模式下 exec,system 等函数调用执行的目录
safe_mode_exec_dir = "/usr/local/sinasrv/php/bin/"
安全模式下 可以修改的、新设置的环境变量名的格式，
说明：下面的设置只允许修改或设置以 PHP_ 为前缀的变量 safe_mode_allowed_env_vars = PHP_
安全模式下 不可修改的环境变量，也就是受保护的不能用 putenv() 重新设置的，
说明：在这里指定的变量，即使用 safe_mode_allowed_env_vars 指定也不能修改。 safe_mode_protected_env_vars = LD_LIBRARY_PATH
默认不允许用 fopen 函数读取打开远程服务器的网页
allow_url_fopen Off
```

##### MySQL

###### 相关目录、文件路径的命名

####### 数据目录命名、配置文件、socket文件命名

：-             |目录/示例                                       |说明/规则
----------------|-----------------------------------------------|---------------------------------
MySQL数据目录根   |/sinasrv/mysql/、 /sinasrv/mysql3308/  |数据根目录前缀 + "mysql+开放端口号"
MySQL库、表数据目录|/sinasrv/mysql/data、 /sinasrv/mysql3308/data|数据根目录前缀 + "mysql+开放端口号"+数据目录
配置文件命名     |/sinasrv/mysql/mysql3310/my3310.cnf |MySQL数据目录根 + "my + 开放端口号 +.cnf"
MySQL socket文件命名| /tmp/mysql3310.sock|/tmp目录名 + "mysql + 开放端口号+.sock"
 
####### 库命名规范
使用域名或者项目名称。比如域名为：news.survey.sina.com.cn，那么库命名为：news_survey 比如项目名称为：通用测试项目，那么库命名为：generaltest
####### 读写帐户命名规范
1.全权账号：默认只配置一个和库同名的帐户，具有该数据库的全部操作权限。

2.可写账号：默认只配置一个可写账护，可以访问该数据库的全部数据，可以insert、update、delete等非管理操作，命名为：库名+"_w" 后缀

3.只读帐户：默认只配置一个只读帐户，可以访问该数据库的全部数据，命名为： 库名 + "_r" 后缀

####### server-id命名规范
使用所在服务器的外网IP地址后两位 + MySQL监听端口号, IP地址不足3位补0， 例如：10.44.6.17，端口为 3306，那么 server-id = 0060173306

######默认配置
MySQL 默认使用 my-large.cnf 作为配置文件，在此基础之上根据一些实际的环境
需求进行特定性的优化配置。
下面是一些主要的配置：

```
   [mysqld]
   datadir                 = /data2/mysql3308
   port                    = 3308
   socket                  = /tmp/mysql3308.sock
   skip-locking
   key_buffer              = 256M
   max_allowed_packet      = 1M
   table_cache             = 256
   sort_buffer_size        = 1M
   read_buffer_size        = 1M
   myisam_sort_buffer_size = 64M
   thread_cache            = 16
   query_cache_size        = 16M
   thread_concurrency      = 4
   max_connections         = 512
   max_user_connections    = 128
   wait_timeout            = 60
   long_query_time         = 1
   server-id               = 0060173308
   log-bin
```
MySQL 打开 --log-slow-queries 选择用来记录较慢的查许，方便程序性能分析和监控。
设置 long_query_time = 1 ，记录查询时间超出 1 秒的访问。
##### Rsync
###### 相关目录、文件路径的命名

：-             |目录/示例                                       |说明/规则
----------------|-----------------------------------------------|---------------------------------
配置文件          |/etc/rsyncd.conf                               |
验证配置文件       |/etc/rsyncd.secrets                            |

注：默认使用虚拟主机的域名为模块名（大写，下划线分割）。 如： news.survey.sina.com.cn 则配模块 NEWS_SURVERY
###### 编译规范
###### 默认配置






目前希望设置为 可执行的目录不能写，可写的目录不能执行，但是如果可写的目录下的
文件被渗透，包含有恶意的或者造成XSS的 javascript 代码，那么理论上还是会存在 威胁，比如用户的 session 信息，或者用户名、密码暂存在 cookie 中的话，那么 仍然存在威胁，因此要考虑用户的cookie中尽量不要包含敏感数据，比如是否可以做到 只包含一个用户的 SessionID
由于在安全模式下设置 include_path 比较麻烦，可能在符号链接中不能生效，或者
php.ini 文件没有包含的路径，在 httpd.conf 包含了也不能用，因此可能不考虑使用 符号链接的方式将 htdocs 目录下不存在的 data, cache 目录链接在 htdocs 目录下， 因为有时得考虑 data, cache 目录下的文件是具有执行权限的。
关于多虚拟主机的 web 站点中各虚拟主机的 htdocs 目录的保护问题。
似乎必须要做保护，要做到各个 vhost 之间不能通过 web 相互浏览彼此的目录，默认 不作任何配置是可以相互浏览的。目前知道的可以做到这个限制的方法是在 php 环境 下，php 有一个叫做 open_basedir 的指令，可以在 httpd.conf 的vhost中设置， 这样可以做到在 php 下每个 vhost 之间不能浏览文件
关于 include_path 的限制问题
前面提到过，我们希望把 web 的目录划分成几个类别，可执行的目录不可写， 可写的目录不可执行，如果可以这样的话就不用考虑 include_path 的问题。 如果不行，那么就会面临这样一个问题。 比如 htdocs 是可以执行的，data, cache 目录是可写的，也可以执行，但 它不再 htdocs 的目录下，是另外一个目录的子目录，但是有符号链接在 htdocs 中，那么如果 data, cache 下的程序需要包含 (include) htdocs 下的文件时， 我们设置了 include_path = ".:/usr/local/sinasrv/lib/php/" 将导致data, cache下的程序不能够包含，将会被拒绝。即使设置如下的方式， 在安全模式下也不可以。 include_path = ".:/usr/local/sinasrv/lib/php/:../include:../../include:../../../include"
关于运行多个站点的 apache 服务器环境的安全问题
前面提到过，关于多个虚拟站点之间默认可以浏览彼此文件的安全隐患， 这里主要是记录一下相关的解决思路。
suexec 模式可以解决 cgi 方式运行的问题，可以参考如下资料
http://httpd.apache.org/docs/suexec.htm
可以考虑在配置好的虚拟站点中放置一个统一命名的虚拟站点配置文件。
比如在 /include/siteconfig.inc 还可以结合 apache 的 setenv 功能直接设置好虚拟站点的配置信息。
常见问题解答





## 四.程序运行环境说明

### 1.程序发布的流程

采用自己开发的文件分发脚本，底层采用rsync做文件传输，使用http协议实现控制命令发送。

如果需要在程序中实现自动的内容发布，有开发的 API 和说明文档提供。

如果在系统命令行下发布，可以使用 distrsync 命令进行发布，单目前这个工具只部署在动态应用架构的服务器中，所以必须要从动态应用的开发服务器上去做程序发布。

目前只提供用 distrsync 进行文件发布，在内部开发的服务器上，经过内部测试后，可以继续用rsync同步到生产环境的测试服务器上测试，通过测试后，则继续用rsync发布到内容后端服务器上，rsync需要用户权限验证。

项目申请后，会同时分配该项目或者该项目开发工程师使用的rsync帐户，会同其他配置说明一起发邮件给申请人。

向生产环境测试的服务器上同步是即时的，同步后就能通过Apache、Nginx服务器访问到。

如果是基于C/C++的程序，需要在测试服务器上编译制作成rpm包格式后，编写好安装说明文档后，一并交给系统部管理员进行统一安装。

### 2.PHP的

### 3.数据库的使用说明

#### 数据库读写分离

数据库使用读写分离的结构，提供用于只写操作(非select的查询)的数据库，和用于只读操作的数据库，因此对于访问量比较大的项目，一定要在开发初期做到数据库的读写分离。

数据库访问目前由应用直接操作实现，暂时还没有好的分布式数据库集群底层组件。

### 4.目录/文件的访问权限和属性设置

Apache、Nginx以www用户和www组用户启动
Apache、Nginx的htdocs目录用户属主为nobody:nobody，只能由root用户写操作，其他用户只读，这里假设没有其他应用程序会使用 nobody 用户帐户。

程序的mmcache,fileupload等目录用户属主为 www:www只能由 root 用户和 www 用户写，其他用户只读。

所有htdocs下的目录都不允许www写操作，只提供专用于写操作的目录，并且不在 htdocs 目录中。

cgi-bin 目录设置同上。


#### chroot环境下的目录属性和文件的访问权限

按照如下方式设置：

确保系统目录都有正确的用户属主

```
   chown -R root:root dev
   chown -R root:root etc
   chown -R root:root lib
   chown -R root:root sbin
   chown -R root:root usr
   chown -R root:root sinasrv/www/htdocs
```

个别目录只能有 root 才能访问

```
   chmod 700 ./{bin,sbin};
   chmod 700 ./usr/{bin,sbin};
   chmod 700 ./usr/local/{bin,sbin}
```

重要的系统配置文件只能由 root 可读写 ，比如 httpd.conf, php.ini, rsyncd.conf

```
chmod 600 /usr/local/sinasrv/apache/conf/httpd.conf chmod 600 /usr/local/sinasrv/lib/php.ini 
```

大多情况下，apache 是以 root 权限启动的，所以下面的目录也设置成仅 root 可访问

```
chmod 700 /usr/local/sinasrv/apache/{bin,conf,libexec,logs}
```

其他的lib目录考虑到php的库和ext模块会再执行期间访问，所以应该是其他用户可以访问的。

#### 关于文件属性

不同扩展名的文件会被强制设置统一的属性，默认情况下都是 644

.cgi 结尾的会设置为 755 属性
.shtml 结尾的如果需要配置 xHackBit 属性，可以设置为 654 属性(g+x)
关于 shtml 的 xHackBit 属性问题，默认不打开，所以默认也不存在
组属性是可执行位的文件
关于 rsync 上传文件时的属性问题，虽然服务器端的 rsyncd.conf
配置文件可以通过设置 uid,gid 来设置文件的属主，但是用户端可以通过 使用 -a 参数来覆盖这个设置。 建议使用这样的参数上传文件：
cd /sinasrv/www/htdocs && rsync -rltD . target::module-name
文件命名说明

#### 库文件存放和命名

应用程序中的库文件放在统一的目录下，在程序的根目录下， 并且命名统一为： include ，库文件的扩展名目前可以考虑 为 *.inc, *.inc.php, *.lib.php
文件命名
不允许有中文名，文件名不允许包含有空格，不允许使用大写字母，应该符合 w3c的网页文件命名规范，即以：0-9,a-z,/. 符号构成。

#### 文件扩展名
c 或者 perl 等其他以 cgi 方式执行的程序，分别用 *.cgi *.pl 扩展名 php 的程序用 *.php, *.php3, *.php4, *.phtml 等作为扩展名
下面扩展名结尾的文件不能从 web 访问
sh, log, txt, sql, bak, bak2, old, 1, 2, php~

### 5.存储系统说明

存储系统由本地存储和网络存储构成，两者都按照如下的原则进行配置：

可执行的不可写
可写的不可执行

存储的访问由应用程序直接操作，目前还没有可用的和底层存储无关的存储组件。


#### cache目录的格式
目前使用的 100x100x100 的三级目录结构非常不利于数据备份，仅空目录就达500M. 建议可以设置更小些的目录，例如目录为两层：32 x 256 ， 每个目录下最大可以有 256 个文件。
#### 实现文件共享的网络存储
目前通过nfs实现多个web前端共享数据，提供的目录类型前面已经提到过。

目录                  | 说明
---------------------|--------------------------------------
ndata, ncache        | 程序可以读写，但不能执行，并且不能从web访问。
nhtdocs              | 程序可以执行，也能从 web 访问，但是不能写。


nfs目录下的文件通过/etc/exports配置文件的anonuid,anongid参数强制设置为
www,www 用户属主，但是文件的属性没有办法控制。

#### 文件系统的优化
数据盘都使用 reiserfs 文件系统，mount 时可以尝试使用的优化参数 nolog mount nfs 的参数为： mount -t nfs -o async,noatime,noexec,nosuid,intr,retry=3,rsize=8192,wsize=8192 muse10i:/data0/nfsexport /nfsdata1
不要使用绝对路径
程序中访问数据目录data，缓存目录cache或者其他目录的文件时，使用相对的磁盘路径， 不使用绝对路径。 可以将绝对路径替换为环境变量，比如在 htdocs 目录下的文件，我们可以通过 DOCUMENT_ROOT环境变量访问，对于 data 目录下的文件，可以使用 SINASRV_DATA_DIR 环境变量访问。
#### 关于慎用nfs cache目录的说明
不推荐使用nfs做数据共享的cache, 如果不是非得共享的cache, 尽量使用本地存储做cache目录，我们可以用cpu的资源损失来换取对i/o的压力 对于实时性要求并不太高的 nfs cache需求，我们可以考虑本地cache + cache 文件同步分发的方案，对于实时性要求较高的需求，我们可以使用MySQL数据库、Memcache、redis+本地cache的方法。
#### 关于数据备份
MySQL 数据库每日备份4次，每隔6小时备份一次。其他数据每日备份1次。 备份数据最短会保留1个月，视存储的空间决定最长保留期。
#### 其他说明

##### 1.程序可以访问的系统命令
默认不提供任何可以访问的系统命令。
chroot 的环境除了必要的几个httpd命令外，其他的命令都不需要。
默认不打开文件上传功能

##### 2.关于邮件的发送功能
目前使用 qmail 进行邮件的发送，监听在本地25端口

##### 管理后台程序的统一管理
提供专用的用于项目后台管理的服务器。

##### 测试服务器
提供专用的用于测试的服务器，除了可以提供用户登陆系统调试外，还打开php的error显示。

##### 虚拟主机的特殊环境变量说明
目前提供下列环境变量

:变量名:                    |:说明:
---------------------------|---------------------
   SINASRV_DATA_DIR        |data 目录
   SINASRV_CACHE_DIR       |cache 目录
   SINASRV_PRIVDATA_DIR    |privdata 目录
   SINASRV_NDATA_DIR       |ndata 目录
   SINASRV_NCACHE_DIR      |ncache 目录
   SINASRV_NHTDOCS_DIR     |nhtdocs 目录
   SINASRV_APPLOGS_DIR     |applogs 目录
   SINASRV_CGIBIN_DIR      |cgi-bin 目录

以后会考虑把数据库的访问帐户也配置成环境变量

公用的函数库

如果项目中使用到的软件库没有安装在我们的服务器中，可以提出申请，会考核软件的特性和安全性，如果没有这方面的问题会统一安装在服务器的/usr/local/sinasrv/lib 目录下。可以是 c/c++ 源代码的库，也可以是 perl/php 等源代码的库。

Apache/Nginx访问日志统计

访问日志每日0点自动归档，生成一个gzip压缩格式的文件。
默认不记录图片日志，错误日志每日0点自动清空一次。
默认所有虚拟主机的访问日志只做归档，并不发送到日志统计部门做分析，如果需要请向系统部提出日志统计申请，系统部会配置实现每日将归档后的日志发送到日志分析部门的日志服务器。 访问日志保留1个月。

##### 在系统后台运行的程序的相关说明

###### 访问 SINASRV_* 变量的说明

项目中，如果需要运行一些在后台执行的程序，比如在cron中执行的程序，或者在系统中运行的程序，这样的程序无法获得设置在Apache/Nginx环境变量中的SINASRV_*变量，为了解决这个问题，做了这样的约定：

项目的后台程序放在 /sinasrv/www/system 目录与项目域名同名的子目录下，如：/sinasrv/www/system/news.survey.sina.com.cn/
在该目录下提供一个名字为： SINASRV_CONFIG 的文本文件

1.这个文件中保存了 SINASRV_* 变量和对应的值，需要用户自己写程序解析出这些变量和值。

2.这个文件的名字和目录格式固定，系统配置升级后自动更新此文件，而不用升级应用程序。
3.后台管理进程中访问 SINASRV_CONFIG 使用相对路径，如： $sinasrv_config = "./SINASRV_CONFIG"

4.后台管理程序中访问其他目录如： htdocs, data, cache 目录，使用绝对路径，通过 SINASRV_* 变量实现，如：$file =$_SINASRV['SINASRV_CACHE_DIR'] . "/20050316/$filename"

这里假设你的程序读取 SINASRV_CONFIG 文件解析出的 SINASRV_* 变量存在名为 $_SINASRV 的数组中。

###### 后台执行程序的执行方式

1.以www或者 nobody 用户在 cron 中定期执行或者驻留在系统中运行。
2.特殊情况下允许以 root 用户运行。
3.程序上传使用 distrsync 方式。
需要说明程序的用途、都要执行什么操作、访问什么资源、执行的方法说明等。
项目初始环境创建后，在文档根目录下生成 system 目录，后台执行的程序都放在
这个目录下。
如果后台程序需要访问cache,data等路径，通过system目录下的SINASRV_CONFIG
配置文件获取路径名，SINASRV_CONFIG 文件是虚拟站点的环境配置文件，包含了 数据路径、缓存路径、数据库等各种配置信息。
其他有待解决的问题

目前希望设置为 可执行的目录不能写，可写的目录不能执行，但是如果可写的目录下的
文件被渗透，包含有恶意的或者造成XSS的 javascript 代码，那么理论上还是会存在 威胁，比如用户的 session 信息，或者用户名、密码暂存在 cookie 中的话，那么 仍然存在威胁，因此要考虑用户的cookie中尽量不要包含敏感数据，比如是否可以做到 只包含一个用户的 SessionID
由于在安全模式下设置 include_path 比较麻烦，可能在符号链接中不能生效，或者
php.ini 文件没有包含的路径，在 httpd.conf 包含了也不能用，因此可能不考虑使用 符号链接的方式将 htdocs 目录下不存在的 data, cache 目录链接在 htdocs 目录下， 因为有时得考虑 data, cache 目录下的文件是具有执行权限的。
关于多虚拟主机的 web 站点中各虚拟主机的 htdocs 目录的保护问题。
似乎必须要做保护，要做到各个 vhost 之间不能通过 web 相互浏览彼此的目录，默认 不作任何配置是可以相互浏览的。目前知道的可以做到这个限制的方法是在 php 环境 下，php 有一个叫做 open_basedir 的指令，可以在 httpd.conf 的vhost中设置， 这样可以做到在 php 下每个 vhost 之间不能浏览文件
关于 include_path 的限制问题
前面提到过，我们希望把 web 的目录划分成几个类别，可执行的目录不可写， 可写的目录不可执行，如果可以这样的话就不用考虑 include_path 的问题。 如果不行，那么就会面临这样一个问题。 比如 htdocs 是可以执行的，data, cache 目录是可写的，也可以执行，但 它不再 htdocs 的目录下，是另外一个目录的子目录，但是有符号链接在 htdocs 中，那么如果 data, cache 下的程序需要包含 (include) htdocs 下的文件时， 我们设置了 include_path = ".:/usr/local/sinasrv/lib/php/" 将导致data, cache下的程序不能够包含，将会被拒绝。即使设置如下的方式， 在安全模式下也不可以。 include_path = ".:/usr/local/sinasrv/lib/php/:../include:../../include:../../../include"
关于运行多个站点的 apache 服务器环境的安全问题
前面提到过，关于多个虚拟站点之间默认可以浏览彼此文件的安全隐患， 这里主要是记录一下相关的解决思路。
suexec 模式可以解决 cgi 方式运行的问题，可以参考如下资料
http://httpd.apache.org/docs/suexec.htm
可以考虑在配置好的虚拟站点中放置一个统一命名的虚拟站点配置文件。
比如在 /include/siteconfig.inc 还可以结合 apache 的 setenv 功能直接设置好虚拟站点的配置信息。
常见问题解答

rsync 时可以删除服务器上的文件嘛？
可以的，rsync时添加一个参数 --delete ，你自己的目录下已经删除的文件就会在服务器起也删除
例如：cd /data1/apache/htdocs/biz.sina.com.cn && rsync --delete -ru . 10.43.6.18::biz.sina.com.cn 如果需要验证的话: cd /data1/apache/htdocs/biz.sina.com.cn && rsync --delete -ru . username@10.43.6.18 ::biz.sina.com.cn
我使用的 dbgateway 开发库服务器不支持，是否可以统一安装？
可以的，请把软件的安装包和安装说明，已经测试程序等发给我，我会制作统一的 软件安装包，进行安装和测试。
我的程序中要使用 php smarty 模板库，请问服务器是否可以统一安装还是我拷贝
在自己的程序目录下？ 如果大家都有需要我会统一安装，统一安装会有几个比较好的地方是节省存储空间， 方便软件库的统一升级管理。
我的程序是用 C 开发的，怎么安装到服务器上去？
你需要制作成二机制的 rpm 包格式后交给系统部服务器管理员， 他们负责统一安装。 如果不会制作 rpm 包，可以提供程序源代码和 Makefile 文件或者其他编译说明， 由系统部负责制作并统一安装。


会议主要内容：
=======================================================================
1。平台架构介绍，
2。目前实现功能介绍
   (自动安装配置/cfengine管理/iptables/sudo/ftp+tls+ldap/chroot apache)
3。目前的文件发布流程
4。如何实现开发环境、测试环境、生产环境 完全分离
5。如何将开发环境的配置、程序实施镜像/同步到测试环境和生产环境
6。如何实现我们的版本控制的功能
7。建立合理的服务器访问控制策略（tcpwapper/iptables/sudo)
8。ftp,ssh 服务的使用
9。目前存在的程序安全问题和处理
10。下一步要做的工作

希望大家提前把目前的问题整理一下，提高讨论的效率！
忘了讨论动态应用平台中的 项目程序代码、共享库代码、js代码、网页目录创建规范

1. 吴申提出：Chroot Apache 中符号链接是否可以使用？
    比如链接htdocs目录下的data目录原本是链接到/data0目录的，chroot 后还能使用吗？
    - 可以的，但是不能使用 link ，而是使用 mount --bind 将命令。
      比如：mount --bind /data0 /data1/chroot/apache/data0
     FreeBSD 有 mount_null 命令实现改功能
      Solaris 有 mount -F lofs 命令实现改功能

 2. 关于程序发布系统和版本控制系统相关问题。
    经过讨论，大家一致认为功能以简单为主，实现如下功能：
    - 单个文件/目录发布
    - 多个文件/目录发布
    - 管理维护程序发布（这类程序主要用于项目中数据的管理或者维护）
    - 管理维护程序的执行属性设置（通过crontab执行还是常住内存执行？什么时间执行？)
    - 版本控制，版本号自动升级
    - 不同版本的备份
    - 发布撤消和旧版本恢复

 3. 陈理捷提出：数据库的 数据结构 如何实现自动备份和版本控制？
    - 这个问题大家意见不一致，部分人认为有的应用中数据结构是不会变的。
   - 我认为这个问题应该考虑，只要有改变就应该算为版本变更，
      需要在项目开发文档和源代码库中体现出来。
      但最终实现这个工作可能有两种方式：
       o 如果数据库有统一的DBA，那么由他实现版本控制。
       o 如果由项目开发人员管理DB，那么他们自己实现版本控制。
    - 数据库的备份目前有统一的备份中心：NetApp R200.

 4. 开发环境是否提供CVS，是否完全通过发布系统实现版本管理？
    - 目前不考虑提供CVS。

 5. 吴申提出：多个项目的 log 和其他数据 的目录如何规范化，标准化？
              目录的创建由谁去完成？
     - 项目开发人员提供配置清单，由系统部管理员来完成配置。

 6. 单独提供一台专做管理使用的服务器，用于项目相关的管理程序的服务。
     - 没问题，项目初期设计时就已经预留了一台管理\监控\log分析\安全审核de服务器。

 7. 实现了程序发布系统后，就能实现开发环境、测试环境、生产环境 3阶段分离。
     在此期间，生产环境的服务器只提供一台开发人员登录维护使用，并做如下授权：
     o 给项目负责人员提供ssh和sudo root的权限，
     o 其他开发人员只提供较低权限的ssh 和 ftp权限。
     o 开发环境也同样授权。

 8. 为了吸取更多的经验，下一个使用动态应用平台的项目按照前面提到的模式进行开发。

 9. 我提出的 由apache自己的安全扩展模块 进行用户输入内容的 转义和过滤，
    大家认为可能会影响程序的运行。
    也许我们应该进行一些测试来进行评估。
目前我对MySQL HA/LB 集群的实现分了如下几块：

1 只读数据库的负载均衡，有多种技术可选，暂时使用4层交换
  其他还有 DNS论询、4层交换、VRRP、ARP Spoof
  * 只实现了4层交换

2 数据库完整备份(每周执行一次)
  * 已经实现

3 数据库增量备份(每小时或者每日)
  X 还未实现

4 数据库快速重建和灾难恢复(读取备份文件)
  * 还未实现

5 数据库健康检查和故障切换
  X 还未实现

6 基于MySQL数据库集群的应用程序中间件程序
  - 实现数据库读库和写库的分离
  - 实现高性能的、优化的数据库查询
  - 实现写库的队列化、读库的Cache化，是否走队列以及Cache是可定制的
  - 隐藏复杂的数据库集群结构，给普通开发人员提供易用的接口程序
