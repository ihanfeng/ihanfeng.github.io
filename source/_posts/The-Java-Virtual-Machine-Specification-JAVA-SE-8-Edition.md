---
title: JAVA虚拟机规范 (JAVA SE 8 Edition)
tags:
  - JAVA
  - 虚拟机
  - JVM
  - JAVA8
categories:
  - JAVA
abbrlink: 35047
date: 2017-10-11 20:00:00
---
# 前言

JAVA9已经发布了，阿里达摩院也横空出世了，在不学习，就跟不上年轻的步伐，就跟不上时代的潮流。作为一个6年的JAVA开发者，是时候深入研究学习。本系列博客，仅作为学习笔记。

  <!-- more -->
# JAVA简史

Java是一种计算机编程语言，拥有跨平台、面向对象、泛型编程的特性，广泛应用于企业级Web应用开发和移动应用开发。

1991年4月，由James Gosling博士领导的绿色计划（Green Project）开始启动，此计划的目的是开发一种能够在各种消费性电子产品（如机顶盒、冰箱、收音机等）上运行的程序架构。这个计划的产品就是Java语言的前身：Oak（橡树）。Oak当时在消费品市场上并不算成功，但随着1995年互联网潮流的兴起，Oak迅速找到了最适合自己发展的市场定位并蜕变成为Java语言。

1992年3月，由于Oak已被用作另一种已存在的编程语言名称，因此必须选一个新的名字——它就是Java，灵感来源于咖啡。

1993年2月，电视机顶盒，FirstPerson试图从时代华纳获得一个电视机顶盒交互系统的一揽子订单。在那时，由于绿色计划不是很成功，随即失去了时代华纳的订单。于是开发的重心从家庭消费电子产品转到了电视盒机顶盒的相关平台上。
　　
1995年5月23日，Oak语言改名为Java，并且在SunWorld大会上正式发布Java 1.0版本。Java语言第一次提出了“Write Once，Run Anywhere”的口号。

1996年1月23日，JDK 1.0发布，Java语言有了第一个正式版本的运行环境。JDK 1.0提供了一个纯解释执行的Java虚拟机实现（Sun Classic VM）。JDK 1.0版本的代表技术包括：Java虚拟机、Applet、AWT等。

1996年4月，10个最主要的操作系统供应商申明将在其产品中嵌入Java技术。同年9月，已有大约8.3万个网页应用了Java技术来制作。在1996年5月底，Sun公司于美国旧金山举行了首届JavaOne大会，从此JavaOne成为全世界数百万Java语言开发者每年一度的技术盛会。

1997年2月19日，Sun公司发布了JDK 1.1，Java技术的一些最基础的支撑点（如JDBC等）都是在JDK 1.1版本中发布的，JDK 1.1版的技术代表有：JAR文件格式、JDBC、JavaBeans、RMI。Java语法也有了一定的发展，如内部类（Inner Class）和反射（Reflection）都是在这个时候出现的。

直到1999年4月8日，JDK 1.1一共发布了1.1.0～1.1.8九个版本。从1.1.4之后，每个JDK版本都有一个自己的名字（工程代号），分别为：JDK 1.1.4 - Sparkler（宝石）、JDK 1.1.5 - Pumpkin（南瓜）、JDK 1.1.6 - Abigail（阿比盖尔，女子名）、JDK 1.1.7 - Brutus（布鲁图，古罗马政治家和将军）和JDK 1.1.8 – Chelsea（切尔西，城市名）。

1998年12月4日，JDK迎来了一个里程碑式的版本JDK 1.2，工程代号为Playground（竞技场），Sun在这个版本中把Java技术体系拆分为3个方向，分别是面向桌面应用开发的J2SE（Java 2 Platform， Standard Edition）、面向企业级开发的J2EE（Java 2 Platform， Enterprise Edition）和面向手机等移动终端开发的J2ME（Java 2 Platform， Micro Edition）。在这个版本中出现的代表性技术非常多，如EJB、Java Plug-in、Java IDL、Swing等，并且这个版本中Java虚拟机第一次内置了JIT（Just In Time）编译器（JDK 1.2中曾并存过3个虚拟机，Classic VM、HotSpot VM和Exact VM，其中Exact VM只在Solaris平台出现过；后面两个虚拟机都是内置JIT编译器的，而之前版本所带的Classic VM只能以外挂的形式使用JIT编译器）。在语言和API级别上，Java添加了strictfp关键字与现在Java编码之中极为常用的一系列Collections集合类。

在1999年3月和7月，分别有JDK 1.2.1和JDK 1.2.2两个小版本发布。

1999年4月27日，HotSpot虚拟机发布，HotSpot最初由一家名为“Longview Technologies”的小公司开发，因为HotSpot的优异表现，这家公司在1997年被Sun公司收购了。HotSpot虚拟机发布时是作为JDK 1.2的附加程序提供的，后来它成为了JDK 1.3及之后所有版本的Sun JDK的默认虚拟机。

2000年5月8日，工程代号为Kestrel（美洲红隼）的JDK 1.3发布，JDK 1.3相对于JDK 1.2的改进主要表现在一些类库上（如数学运算和新的Timer API等），JNDI服务从JDK 1.3开始被作为一项平台级服务提供（以前JNDI仅仅是一项扩展），使用CORBA IIOP来实现RMI的通信协议，等等。这个版本还对Java 2D做了很多改进，提供了大量新的Java 2D API，并且新添加了JavaSound类库。JDK 1.3有1个修正版本JDK 1.3.1，工程代号为Ladybird（瓢虫），于2001年5月17日发布。

自从JDK 1.3开始，Sun维持了一个习惯：大约每隔两年发布一个JDK的主版本，以动物命名，期间发布的各个修正版本则以昆虫作为工程名称。

2002年2月13日，JDK 1.4发布，工程代号为Merlin（灰背隼）。JDK 1.4是Java真正走向成熟的一个版本，Compaq、Fujitsu、SAS、Symbian、IBM等著名公司都有参与甚至实现自己独立的JDK 1.4。哪怕是在十多年后的今天，仍然有许多主流应用（Spring、Hibernate、Struts等）能直接运行在JDK 1.4之上，或者继续发布能运行在JDK 1.4上的版本。JDK 1.4同样发布了很多新的技术特性，如正则表达式、异常链、NIO、日志类、XML解析器和XSLT转换器等。

JDK 1.4有两个后续修正版：

  2002年9月16日发布的工程代号为Grasshopper（蚱蜢）的JDK 1.4.1

  2003年6月26日发布的工程代号为Mantis（螳螂）的JDK 1.4.2。

2002年前后还发生了一件与Java没有直接关系，但事实上对Java的发展进程影响很大的事件，那就是微软公司的.NET Framework发布了。这个无论是技术实现上还是目标用户上都与Java有很多相近之处的技术平台给Java带来了很多讨论、比较和竞争，.NET平台和Java平台之间声势浩大的孰优孰劣的论战到目前为止都在继续。

2004年9月30日，JDK 1.5发布，工程代号Tiger（老虎）。从JDK 1.2以来，Java在语法层面上的变换一直很小，而JDK 1.5在Java语法易用性上做出了非常大的改进。例如，自动装箱、泛型、动态注解、枚举、可变长参数、遍历循环（foreach循环）等语法特性都是在JDK 1.5中加入的。在虚拟机和API层面上，这个版本改进了Java的内存模型（Java Memory Model，JMM）、提供了java.util.concurrent并发包等。另外，JDK 1.5是官方声明可以支持Windows 9x平台的最后一个JDK版本。

2006年12月11日，JDK 1.6发布，工程代号Mustang（野马）。在这个版本中，Sun终结了从JDK 1.2开始已经有8年历史的J2EE、J2SE、J2ME的命名方式，启用Java SE 6、Java EE 6、Java ME 6的命名方式。JDK 1.6的改进包括：提供动态语言支持（通过内置Mozilla Java Rhino引擎实现）、提供编译API和微型HTTP服务器API等。同时，这个版本对Java虚拟机内部做了大量改进，包括锁与同步、垃圾收集、类加载等方面的算法都有相当多的改动。

在2006年11月13日的JavaOne大会上，Sun公司宣布最终会将Java开源，并在随后的一年多时间内，陆续将JDK的各个部分在GPL v2（GNU General Public License v2）协议下公开了源码，并建立了OpenJDK组织对这些源码进行独立管理。除了极少量的产权代码（Encumbered Code，这部分代码大多是Sun本身也无权限进行开源处理的）外，OpenJDK几乎包括了Sun JDK的全部代码，OpenJDK的质量主管曾经表示，在JDK 1.7中，Sun JDK和OpenJDK除了代码文件头的版权注释之外，代码基本上完全一样，所以OpenJDK 7与Sun JDK 1.7本质上就是同一套代码库开发的产品。

JDK 1.6发布以后，由于代码复杂性的增加、JDK开源、开发JavaFX、经济危机及Sun收购案等原因，Sun在JDK发展以外的事情上耗费了很多资源，JDK的更新没有再维持两年发布一个主版本的发展速度。JDK 1.6到目前为止一共发布了37个Update版本，最新的版本为Java SE 6 Update 37，于2012年10月16日发布。

2009年2月19日，工程代号为Dolphin（海豚）的JDK 1.7完成了其第一个里程碑版本。根据JDK 1.7的功能规划，一共设置了10个里程碑。最后一个里程碑版本原计划于2010年9月9日结束，但由于各种原因，JDK 1.7最终无法按计划完成。

从JDK 1.7最开始的功能规划来看，它本应是一个包含许多重要改进的JDK版本，其中的Lambda项目（Lambda表达式、函数式编程）、Jigsaw项目（虚拟机模块化支持）、动态语言支持、GarbageFirst收集器和Coin项目（语言细节进化）等子项目对于Java业界都会产生深远的影响。在JDK 1.7开发期间，Sun公司由于相继在技术竞争和商业竞争中都陷入泥潭，公司的股票市值跌至仅有高峰时期的3%，已无力推动JDK 1.7的研发工作按正常计划进行。为了尽快结束JDK 1.7长期“跳票”的问题，Oracle公司收购Sun公司后不久便宣布将实行“B计划”，大幅裁剪了JDK 1.7预定目标，以便保证JDK 1.7的正式版能够于2011年7月28日准时发布。“B计划”把不能按时完成的Lambda项目、Jigsaw项目和Coin项目的部分改进延迟到JDK 1.8之中。最终，JDK 1.7的主要改进包括：提供新的G1收集器（G1在发布时依然处于Experimental状态，直至2012年4月的Update 4中才正式“转正”）、加强对非Java语言的调用支持（JSR-292，这项特性到目前为止依然没有完全实现定型）、升级类加载架构等。

到目前为止，JDK 1.7已经发布了9个Update版本，最新的Java SE 7 Update 9于2012年10月16日发布。从Java SE 7 Update 4起，Oracle开始支持Mac OS X操作系统，并在Update 6中达到完全支持的程度，同时，在Update 6中还对ARM指令集架构提供了支持。至此，官方提供的JDK可以运行于Windows（不含Windows 9x）、Linux、Solaris和Mac OS平台上，支持ARM、x86、x64和Sparc指令集架构类型。

2009年4月20日，Oracle公司宣布正式以74亿美元的价格收购Sun公司，Java商标从此正式归Oracle所有（Java语言本身并不属于哪间公司所有，它由JCP组织进行管理，尽管JCP主要是由Sun公司或者说Oracle公司所领导的）。由于此前Oracle公司已经收购了另外一家大型的中间件企业BEA公司，在完成对Sun公司的收购之后，Oracle公司分别从BEA和Sun中取得了目前三大商业虚拟机的其中两个：JRockit和HotSpot，Oracle公司宣布在未来1～2年的时间内，将把这两个优秀的虚拟机互相取长补短，最终合二为一。可以预见在不久的将来，Java虚拟机技术将会产生相当巨大的变化。

2011年7月28日，Oracle公司发布Java SE 7

2014年3月18日，Oracle公司发表Java SE 8

2017年9月22日，Oracle公司发表Java SE 9

# JAVA虚拟机

JAVA虚拟机是整个JAVA平台的基石，是JAVA技术用以实现硬件无关与操作系统无关的关键部分，是Java语言生成出极小体积的编译代码的运行平台，是保障用户机器免于恶意代码损害的屏障。

JAVA虚拟机可以看做是一台抽象的计算，有自己的指令集以及各种运行时内存区域。

JAVA虚拟机与JAVA语言并没有必然的联系，它只与特定的二进制文件格式--class文件格式所关联。class文件包含了JAVA虚拟机指令集和符号表，以及其他的一些辅助信息。

基于安全方面的考虑，Java虚拟机在class文件中施加了许多强制性的语法和结构化约束，凡是能用class文件正确表达出来的编程语言，都可以放在Java虚拟机里面执行。由于她是一个通用的、与机器无关的执行平台，所以其他语言的实现者都可以考虑将Java虚拟机作为那些语言的交付媒介。


# JAVA学习目录

官方学习教程：[官方指南](http://docs.oracle.com/javase/specs/)

1. [JAVA虚拟机结构](http://zhujunwang.cn/2017/10/11/The-Structure-of-the-Java-Virtual-Machine/)
