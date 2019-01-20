---
layout:     post
title:      "Welcome to Haibin Blog"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-01-20 20:25:33
author:     "linhaibin961"
header-img: "img/post-bg-2019.jpg"
tags:
    - 生活
---

# java内存溢出记录

标签（空格分隔）： java 内存溢出

---
##*ps -ef |grep java*
下面给出了一个简单的内存泄露的例子。在这个例子中，我们循环申请Object对象，并将所申请的对象放入一个Vector中，如果我们仅仅释放引用本身，那么Vector仍然引用该对象，所以这个对象对GC来说是不可回收的。因此，如果对象加入到Vector后，还必须从Vector中删除，最简单的方法就是将Vector对象设置为null。
```java
public class test{

public static void main (String[] args)
	
	Vector v=new Vector(10);
	for (int i=1;i<100; i++)
	{
		Object o=new Object();
		v.add(o);
		o=null;	
	}
}
```
//此时，所有的Object对象都没有被释放，因为变量v引用这些对象。
##解决方案

linux环境下：
打开在Tomcat的安装目录的bin文件的catalina.sh文件,进入编辑状态.
在注释后面加上如下脚本:
```
JAVA_OPTS='-Xms2048m -Xmx4096m'
JAVA_OPTS="$JAVA_OPTS -server -XX:PermSize=128M -XX:MaxPermSize=512m"

//配置jconsole
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.port=60001"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
//第二种
java -cp . -Dcom.sun.management.jmxremote.port=60001 -Dcom.sun.managent.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false JConsoleTest
```
其中 JAVA_OPTS='-Xms512m -Xmx1024m' 是设置Tomcat使用的内存的大小.
-XX:PermSize=64M -XX:MaxPermSize=256m 指定类空间(用于加载类)的内存大小 
保存后，重新以命令行的方式运行tomcat，即可，然后通过最后面介绍的如何观察tomcat现有内存情况的方法进行查看是否已经变更成功。


----------


查看现有tomcat的内存大小情况
1、启动tomcat
2、访问 http://localhost:8080/manager/status ,并输入您在安装tomcat时输入的用户与口令，如 admin ，密码 admin（密码是您在tomcat安装时输入的）
注：添加用户，修改conf/tomcat-users.xml
```
<role rolename="manager"/>  
<role rolename="manager-gui"/>  
<role rolename="admin"/>  
<role rolename="admin-gui"/>  
<role rolename="manager-script"/>  
<role rolename="manager-jmx"/>  
<role rolename="manager-status"/>  
<user username="Tomcat" password="Tomcat" roles="admin-gui,admin,manager-gui,manager,manager-script,manager-jmx,manager-status"/>
```
Free memory: 241.80 MB Total memory: 254.06 MB Max memory: 508.06 MB
上面的文字即代表了，当前空闲内存、当前总内存、最大可使用内存三个数据。
确定了最大内存足够大时，tomcat即可正常运转
最后总结下内存设置中常用的几个参数
(1)-Xms，jvm启动时，初始分配的堆/栈内存
(2)-Xmx，JVM最大允许分配的堆/栈内存，按需分配
(3)-Xss，设定每个线程的堆栈大小
(4)-XX:PermSize，JVM初始分配的非堆内存
(5)-XX:MaxPermSize，JVM最大允许分配的非堆内存，按需分配


----------


###推荐的三款内存检测工具
1、jconsole JDK自带的内存监测工具，路径jdk bin目录下jconsole.exe，双击可运行。连接方式有两种，第一种是本地方式如调试时运行的进程可以直接连，第二种是远程方式，可以连接以服务形式启动的进程。远程连接方式是：在目标进程的jvm启动参数中添加-Dcom.sun.management.jmxremote.port=1090 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false 1090是监听的端口号具体使用时要进行修改，然后使用IP加端口号连接即可。通过该工具可以监测到当时内存的大小，CPU的使用量以及类的加载，还提供了手动gc的功能。优点是效率高，速度快，在不影响进行运行的情况下监测产品的运行。缺点是无法看到类或者对象之类的具体信息。使用方式很简单点击几下就可以知道功能如何了，确实有不明白之处可以上网查询文档。

2.VisualVM
VisualVM是一个集成多个JDK命令行工具的可视化工具。VisualVM基于NetBeans平台开发，它具备了插件扩展功能的特性，通过插件的扩展，可用于显示虚拟机进程及进程的配置和环境信息(jps，jinfo)，监视应用程序的CPU、GC、堆、方法区及线程的信息(jstat、jstack)等。VisualVM在JDK/bin目录下。

3、JProfiler 收费的工具，但是到处都有破解办法。安装好以后按照配置调试的方式配置好一个本地的session即可运行。可以监测当时的内存、CPU、线程等，能具体的列出内存的占用情况，还可以就某个类进行分析。优点很多，缺点太影响速度，而且有的类可能无法被织入方法，例如我使用jprofiler时一直没有备份成功过，总会有一些类的错误

----------