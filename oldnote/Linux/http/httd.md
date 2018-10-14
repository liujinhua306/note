[TOC]



# httpd的由来



httpd早期是由NCSA开发的，但是后来这个项目解散，成员被散落在各个公司，但是这些开发人员非常喜欢这个软件，于是自发的通过互联网协作，继续为这个软件新增许多功能，包括打补丁，新增功能，后来人们就戏称这个httpd为充满了补丁的服务，A Patchy Server ,简写为apache



ASF:Apache Software Foundation:Apache软件基金会

​	web:httpd 很多时候，我们以Apache来称呼httpd，那是因为早期httpd就是以apache server，因此早期我们来说Apache的时候，就是说的是Apache的web项目httpd，但是到今天为止Apache已经有了很多的开源软件，httpd只是其中的一个而已，所以Apache的很多顶级项目都可以这样来访问:httpd.apache.org/,tomcat.apache.org， flume.apache.org，hadoop.apache.org 等等



# httpd安装配置



* 事先创建进程
* 按需维持适当的进程
* 模块化设计，核心比较小，各种功能都可以模块的添加，支持运行配置，支持单独编译模块
* 支持多种方式的虚拟主机配置
* 支持https协议（mod_ssl)
* 支持用户认证
* 支持基于IP或者主机名的ACL（访问控制）
* 支持每目录的访问控制（ACL）
* 支持URL重写（重要）





## 虚拟主机

1台物理服务器，web程序也只有一个，却可以服务多个不同的站点



* 基于IP的虚拟主机
* 基于端口的虚拟主机
* 基于域名的虚拟主机（常用）



## 目录说明（配置文件和目录）



```
/usr/sbin/httpd (MPM:prefork) 后台会启动多个进程，除了一个进程的用户时root之外，其他的进程的处理用户都是apache用户，apache组
	httpd: root，root	#(master process)主导进程，用来创建进程，销毁进程
	httpd: apache, apache	#(worker process)

/etc/rc.d/init.d/httpd	#配置文件

Port:(80/tcp), ssl:(443/tcp) #开启的端口

/etc/httpd #工作的根目录，进程运行的根目录，相当于程序安装的目录

/etc/httpd/conf 	#配置文件目录
	主配置文件：httpd.conf	#主配置文件
	/etc/httpd/conf.d/*.conf	也是上面主配置文件的组成部分

/etc/httpd/modules: #模块目录

/etc/httpd/logs	--> /var/log/httpd	#日志目录
	日志文件有两类：访问日志:access_log ，错误日志：err_log
	
/var/www/
	html 静态页面所在的目录
	cgi-bin	动态内容的目录，cgi是一种能够让web服务器和应用程序通信的一种机制，让应用程序启动启动额外的程序处理动态内容
	cgi:Common Gateway Interafce 通用网关接口，让web服务器和应用程序服务器打交道的一种协议
	
	Client-->httpd(index.cgi)-->Spawn Process(index.cgi)-->httpd-->Client
	首先由客户端发起httpd请求
	httpd的服务在内部调用一个脚本，如index.cgi
	cgi启动一个和脚本对应的应用程序服务，该应用程序执行该脚本(index.cgi)
	应用程序将脚本的执行结果返回给httpd,最终结果由httpd返回给客户端
	
	
	对于动态的脚本而言，每来一个请求，都会开启一个新的进程，这样如果请求的动态内容的客户端很多的话，那么后台就会有很多的进程，解决的方式是在后台启动一个进程管理所有的需要新开的处理动态内容的进程，由这个进程负责维护所有的动态进程，这就叫做fastcgi(适用于PHP)
```

![image-20181014141836894](/Users/chenyansong/Documents/note/images/http/fastcgi.png)



应用程序服务器进程和web服务器进程通过某个socket进程通信，这样应用程序服务进程和web服务器进程可以放在不同的服务器上，这样就可以实现分离，这样就可以实现这样的场景，一个服务器实现静态内容的访问，另一个服务器实现的是动态内容的应用程序服务；当一个用户请求的是动态内容的时候，可以通过本地的网络来发送给另外一台应用程序主机，这也就是前后端分离的结果



![image-20181014144139576](/Users/chenyansong/Documents/note/images/http/fastcgi2.png)



## 安装

```
yum -y install httpd

rpm -ql httpd
```

![image-20181014175755949](/Users/chenyansong/Documents/note/images/http/httpd.png)



![image-20181014180042314](/Users/chenyansong/Documents/note/images/http/httpd2.png)



会启动一个root用户的进程，然后启动多个Apache的用户进程

![image-20181014180217355](/Users/chenyansong/Documents/note/images/http/httpd3.png)



## 配置文件说明



httpd配置文件

```
vim /etc/httpd/httd.conf

directive			vlaue
指令(不区分大小写)	   值(分区大小写)
```



指令的查询

Httpd.apache.org下面进入到Documents页面，找到“指令快速参考”

![image-20181014183412142](/Users/chenyansong/Documents/note/images/http/httpd-config1.png)

选择指令的其实字母，能够快速查询

![image-20181014183514433](/Users/chenyansong/Documents/note/images/http/httpd-config2.png)



```
#httpd的根目录
ServerRoot	"/etc/httpd"

#pid的保存目录，相对于ServerRoot
PidFile	run/httpd.pid

#tcp的timeout
Timeout 120


#是否使用长链接
KeepAlive Off

#
MaxKeepAliveRequests	100
KeepAliveTimeout	15

#prefork模式
<IfModule prefork.c>
StartServers 8	#启动的时候启动多少个空闲进程(一个root的用户进程；7个apache的用户进程)
MinSpareServers  5	#最小空闲进程
MaxSpareServers		20	#最大空闲进程
ServerLimit	256		#MaxClients的最大值
MaxClients	256		#
MaxRequestsPerChild	4000	#每一个进程的最大响应的请求次数
</IfModule>


#worker模式
<IfModule worker.c>
StartServers 2	#默认启动多少个进程
MaxClients	150	#MaxClients的最大值
MinSpareThreads	25	#最小空闲线程(所有进程加起来)
MaxSpareThreads	75	#最大空闲线程(所有进程加起来)
ThreadsPerChild	25	#每个进程可以生成多少个线程
MaxRequestsPerChild	0	#每一个进程的最多响应的请求次数，0表示不做限定
</IfModule>

Listen	80	#监听的端口
Listen	8080

#LoadModule	foo_module(模块名称)  modules/mod_auth_basic.so(模块路径)
LoadModule	auth_basic_module  modules/mod_auth_basic.so

#包含的配置文件的目录
Include conf.d/*.conf

#worker进程需要使用普通用户运行(这里指定是谁)
User	apache	
Group	apache
```



MPM:multi Path Modules(多处理模块)，定义多用户请求的时候，响应的模型，常用的方式有：

* mpm_winnt, 
* prefork(预先生成进程，一个请求用一个进程响应)
* worker(基于线程来工作：一个请求用一个线程响应；启动多个进程，每个进程生成多个线程，每一个线程响应一个请求)
* event(基于事件驱动，一个进程处理多个用户请求，httpd2.4之后支持，nginx默认使用这种)



```
#这个是httpd服务的配置文件，可以指定启动的是以prefork，worker,event的方式工作
vim /etc/sysconfig/httpd 

#可以改成对应的模型
/usr/sbin/httpd.worker
/usr/sbin/httpd.event
```




