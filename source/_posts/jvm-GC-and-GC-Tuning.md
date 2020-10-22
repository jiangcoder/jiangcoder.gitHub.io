title: Git如何回退版本
toc: true
categories: java
tags:
  - java
thumbnail: /logo/java.jpg
---
<div id="article_content" class="article_content clearfix">
            <link rel="stylesheet" href="https://csdnimg.cn/release/phoenix/template/css/ck_htmledit_views-833878f763.css">
                                        <link rel="stylesheet" href="https://csdnimg.cn/release/phoenix/template/css/ck_htmledit_views-833878f763.css">
                <div class="htmledit_views" id="content_views">
                                            <p id="main-toc"><strong>目录</strong></p>

<p id="1%E3%80%81%E4%BB%80%E4%B9%88%E6%98%AF%E5%9E%83%E5%9C%BE-toc" style="margin-left:0px;"><a href="#1%E3%80%81%E4%BB%80%E4%B9%88%E6%98%AF%E5%9E%83%E5%9C%BE" rel="nofollow" target="_self">1、什么是垃圾</a></p>

<p id="2%E3%80%81%E5%A6%82%E4%BD%95%E5%AE%9A%E4%BD%8D%E5%9E%83%E5%9C%BE-toc" style="margin-left:0px;"><a href="#2%E3%80%81%E5%A6%82%E4%BD%95%E5%AE%9A%E4%BD%8D%E5%9E%83%E5%9C%BE" rel="nofollow" target="_self">2、如何定位垃圾</a></p>

<p id="reference%20count-toc" style="margin-left:40px;"><a href="#reference%20count" rel="nofollow" target="_self">reference count</a></p>

<p id="Root%20Searching-toc" style="margin-left:40px;"><a href="#Root%20Searching" rel="nofollow" target="_self">Root Searching</a></p>

<p id="3%E3%80%81%E5%B8%B8%E8%A7%81%E7%9A%84%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%AE%97%E6%B3%95-toc" style="margin-left:0px;"><a href="#3%E3%80%81%E5%B8%B8%E8%A7%81%E7%9A%84%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%AE%97%E6%B3%95" rel="nofollow" target="_self">3、常见的垃圾回收算法</a></p>

<p id="Mark-Sweep(%E6%A0%87%E8%AE%B0%E6%B8%85%E9%99%A4)-toc" style="margin-left:40px;"><a href="#Mark-Sweep(%E6%A0%87%E8%AE%B0%E6%B8%85%E9%99%A4)" rel="nofollow" target="_self">Mark-Sweep(标记清除)</a></p>

<p id="Copying(%E6%8B%B7%E8%B4%9D%E7%AE%97%E6%B3%95%EF%BC%89-toc" style="margin-left:40px;"><a href="#Copying(%E6%8B%B7%E8%B4%9D%E7%AE%97%E6%B3%95%EF%BC%89" rel="nofollow" target="_self">Copying(拷贝算法）</a></p>

<p id="Mark-Compact%EF%BC%88%E6%A0%87%E8%AE%B0%E5%8E%8B%E7%BC%A9)-toc" style="margin-left:40px;"><a href="#Mark-Compact%EF%BC%88%E6%A0%87%E8%AE%B0%E5%8E%8B%E7%BC%A9)" rel="nofollow" target="_self">Mark-Compact（标记压缩)</a><a href="#%E2%80%8B" rel="nofollow" target="_self">​</a></p>

<p id="4%E3%80%81JVM%E5%86%85%E5%AD%98%E5%88%86%E4%BB%A3%E6%A8%A1%E5%9E%8B(%E7%94%A8%E4%BA%8E%E5%88%86%E4%BB%A3%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%AE%97%E6%B3%95%EF%BC%89-toc" style="margin-left:0px;"><a href="#4%E3%80%81JVM%E5%86%85%E5%AD%98%E5%88%86%E4%BB%A3%E6%A8%A1%E5%9E%8B(%E7%94%A8%E4%BA%8E%E5%88%86%E4%BB%A3%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%AE%97%E6%B3%95%EF%BC%89" rel="nofollow" target="_self">4、JVM内存分代模型(用于分代垃圾回收算法）</a></p>

<p id="%E5%A0%86%E5%86%85%E5%AD%98%E9%80%BB%E8%BE%91%E5%88%86%E5%8C%BA-toc" style="margin-left:40px;"><a href="#%E5%A0%86%E5%86%85%E5%AD%98%E9%80%BB%E8%BE%91%E5%88%86%E5%8C%BA" rel="nofollow" target="_self">堆内存逻辑分区</a></p>

<p id="5%E3%80%81%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8-toc" style="margin-left:0px;"><a href="#5%E3%80%81%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8" rel="nofollow" target="_self">5、垃圾回收器</a></p>

<p id="Serial-toc" style="margin-left:40px;"><a href="#Serial" rel="nofollow" target="_self">Serial</a></p>

<p id="Parallel%20Scavenge-toc" style="margin-left:40px;"><a href="#Parallel%20Scavenge" rel="nofollow" target="_self">Parallel Scavenge</a></p>

<p id="ParNew-toc" style="margin-left:40px;"><a href="#ParNew" rel="nofollow" target="_self">ParNew</a></p>

<p id="SerialOld-toc" style="margin-left:40px;"><a href="#SerialOld" rel="nofollow" target="_self">SerialOld</a></p>

<p id="Parallel%20Old-toc" style="margin-left:40px;"><a href="#Parallel%20Old" rel="nofollow" target="_self">Parallel Old</a></p>

<p id="CMS-toc" style="margin-left:40px;"><a href="#CMS" rel="nofollow" target="_self">CMS</a></p>

<p id="6%E3%80%81JVM%E5%8F%82%E6%95%B0-toc" style="margin-left:0px;"><a href="#6%E3%80%81JVM%E5%8F%82%E6%95%B0" rel="nofollow" target="_self">6、JVM参数</a></p>

<p id="7%E3%80%81%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE-toc" style="margin-left:0px;"><a href="#7%E3%80%81%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE" rel="nofollow" target="_self">7、思维导图</a></p>

<hr id="hr-toc"><h1 id="1%E3%80%81%E4%BB%80%E4%B9%88%E6%98%AF%E5%9E%83%E5%9C%BE"><a name="t0"></a><a name="t0"></a>1、什么是垃圾</h1>

<p>内存分配与回收方式：</p>

<ul><li>
	<p>C语言：malloc、free</p>
	</li>
	<li>
	<p>C++：new、delete</p>
	</li>
	<li>
	<p>Java：new&nbsp; 自动回收内存</p>
	</li>
</ul><p>自动回收内存系统不容易出错，手动回收内存，容易出现以下的错误：</p>

<ul><li>
	<p>忘记回收</p>
	</li>
	<li>
	<p>多次回收</p>
	</li>
</ul><p>垃圾的定义：没有任何引用指向的一个对象或者多个对象(循环引用）。</p>

<p><img alt="" height="263" src="https://img-blog.csdnimg.cn/20200218153041603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="489"></p>

<p>当把成员变量设置为空(null)之后，不再指向任何引用对象，那么该对象就被称作垃圾：</p>

<h2 id="%E2%80%8B"><a name="t1"></a><img alt="" height="259" src="https://img-blog.csdnimg.cn/20200218154813459.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="503"></h2>

<p>还有一种情况，多个对象之间互相引用，但是没有其他的引用指向这个循环的对象。&nbsp;&nbsp;</p>

<h2><a name="t2"></a><img alt="" height="320" src="https://img-blog.csdnimg.cn/20200218155228678.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="503"></h2>

<h1 id="2%E3%80%81%E5%A6%82%E4%BD%95%E5%AE%9A%E4%BD%8D%E5%9E%83%E5%9C%BE"><a name="t3"></a><a name="t3"></a>2、如何定位垃圾</h1>

<ul><li>
	<h2 id="reference%20count"><a name="t4"></a><a name="t4"></a>reference count</h2>
	</li>
</ul><h2><a name="t5"></a><img alt="" height="381" src="https://img-blog.csdnimg.cn/20200218160722764.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="508"></h2>

<p>当引用计数变为0的时候，这个对象就成为垃圾了。但是引用计数不能解决对象循环引用</p>

<p>如下：</p>

<p><img alt="" height="320" src="https://img-blog.csdnimg.cn/20200218155228678.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="503"></p>

<p>每个引用计数都是1，但是它们全部是垃圾，所以用引用计数的方式的话，这些垃圾就找不到了，会发生内存泄漏。</p>

<ul><li>
	<h2 id="Root%20Searching"><a name="t6"></a><a name="t6"></a>Root Searching</h2>
	</li>
</ul><p>根可达或者根搜索算法。</p>

<p>通过程序找到一些根对象，通过根对象找到它所连接的那些对象不是垃圾，其他的都是垃圾。</p>

<p>Roots：线程栈变量、静态变量、常量池、JNI指针</p>

<h2><a name="t7"></a><img alt="" height="334" src="https://img-blog.csdnimg.cn/20200218163811236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="684"></h2>

<h1 id="3%E3%80%81%E5%B8%B8%E8%A7%81%E7%9A%84%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%AE%97%E6%B3%95"><a name="t8"></a><a name="t8"></a>3、常见的垃圾回收算法</h1>

<ul><li>
	<h2 id="Mark-Sweep(%E6%A0%87%E8%AE%B0%E6%B8%85%E9%99%A4)"><a name="t9"></a><a name="t9"></a>Mark-Sweep(标记清除)</h2>
	</li>
</ul><p><img alt="" height="461" src="https://img-blog.csdnimg.cn/2020021817055046.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="698"></p>

<p>将可回收的对象标记为非垃圾。</p>

<p>缺点：位置不连续，产生内存碎片。</p>

<ul><li>
	<h2 id="Copying(%E6%8B%B7%E8%B4%9D%E7%AE%97%E6%B3%95%EF%BC%89"><a name="t10"></a><a name="t10"></a>Copying(拷贝算法）</h2>
	</li>
</ul><p><img alt="" height="409" src="https://img-blog.csdnimg.cn/20200218171626281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="697"></p>

<p>内存一分为二，将存活对象复制到未使用的内存中，原内存全部标记为可使用；新分配内存时先分配存活对象所在的那段内存，垃圾回收时，重复上述操作。</p>

<p>特点：没有碎片，但是内浪费空间。最大的问题：内存浪费。</p>

<ul><li>
	<h2 id="Mark-Compact%EF%BC%88%E6%A0%87%E8%AE%B0%E5%8E%8B%E7%BC%A9)"><a name="t11"></a><a name="t11"></a>Mark-Compact（标记压缩)</h2>
	</li>
</ul><h2><a name="t12"></a><img alt="" height="460" src="https://img-blog.csdnimg.cn/20200218174827773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="691"></h2>

<p>将存活对象依次复制到垃圾对象和未使用的区域中，结合了标记清除和拷贝的做法，但是效率比copy略低。</p>

<p>三种方法找垃圾的效率是一致的，区别在于找到垃圾后对其进行整理的方式。拷贝算法是内存拷贝，是线性地址的拷贝，速度很快的，效率很高。但是压缩算法却不这么简单，因为任意一个内存进行移动时，如果是多线程， 都要进行线程同步；如果是单线程，那单线程的效率本来就低。 所以任何一块内存挪动都要进行线程同步，所以效率肯定是很低的。</p>

<h1 id="4%E3%80%81JVM%E5%86%85%E5%AD%98%E5%88%86%E4%BB%A3%E6%A8%A1%E5%9E%8B(%E7%94%A8%E4%BA%8E%E5%88%86%E4%BB%A3%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%AE%97%E6%B3%95%EF%BC%89"><a name="t13"></a><a name="t13"></a>4、JVM内存分代模型(用于分代垃圾回收算法）</h1>

<p>目前，生产环境中普遍使用的是JDK1.7或JDK1.8，根据JDK版本不同，分代也不同。</p>

<p>JVM中分代：新生代+老年代+永久代(JDK1.7)/元数据区(JDK1.8)Metaspace。</p>

<ul><li>永久代和元数据区是装载Class的，将硬盘上的Class对象load到内存的时候，装载了永久代或者元数据区域，具体放在哪里区别于使用的JDK版本</li>
	<li>永久代必须指定大小限制，而元数据可以设置，也可不设置，无上限(受限于物理内存）</li>
	<li>字符串常量在JDK1.7中，是放在永久代区域；而JDK1.8中，是放在堆里</li>
	<li>MethodArea是一个逻辑概念，并不是指的一个区域，在JDK1.7中对应的就是永久代，JDK1.8中对应的是元数据</li>
</ul><h2 id="%E5%A0%86%E5%86%85%E5%AD%98%E9%80%BB%E8%BE%91%E5%88%86%E5%8C%BA"><a name="t14"></a><a name="t14"></a>堆内存逻辑分区</h2>

<p><img alt="" height="467" src="https://img-blog.csdnimg.cn/20200220122652988.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="817"></p>

<ul><li>新生代中分了两类区域，eden和survivor，而survivor有两块。默认的比例，新生代：老年代=1：3，新生代中eden: survivor：survivor = 8:1:1。</li>
</ul><p>之所以新生代中按照这个比例分配，是因为eden区在GC的时候，90%的对象都会被回收，剩下的存活对象在survivor区是可以放下的。</p>

<p>当创建一个对象时，默认会去找eden区， 如果对象特别大，eden区装不下则直接进入老年代。</p>

<p><span style="color:#f33b45;"><strong>新生代 = Eden + 2个survivor区(survivor0、survivor1）</strong>：</span></p>

<ul><li>YGC(Young GC)回收后，大多数的对象会被回收，活着的对象进入survivor0</li>
	<li>再次YGC，活着的对象eden+s0拷贝到s1，将eden和s0清空</li>
	<li>再次YGC，活着的对象eden+s1拷贝到s0，将eden和s1清空</li>
	<li>年龄足够-&gt;老年代(年龄足够：15，CMS 6)</li>
	<li>survivor区装不下的时候，装不下的部分直接进入老年代</li>
</ul><p><span style="color:#f33b45;"><strong>老年代：</strong></span></p>

<ul><li>顽固份子</li>
	<li>老年代区域满了，就进行Full GC(简称FGC, FGC包括新生代和老年代同时GC)</li>
</ul><p><span style="color:#f33b45;"><strong>GC Tuning：</strong></span>尽量减少FGC。</p>

<h1 id="5%E3%80%81%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8"><a name="t15"></a><a name="t15"></a>5、垃圾回收器</h1>

<p><img alt="" height="414" src="https://img-blog.csdnimg.cn/20200220125708117.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="863"></p>

<ul><li>Serial、ParNew、Parallel Scavenge是用于回收Young Generation</li>
	<li>CMS、Serial Old、Parallel Old是用于回收Old Generation</li>
	<li>G1、ZGC、Shenandoah不区分老年代和新生代。</li>
	<li>Epsilon是一个空的GC，仅仅用于调试JDK。</li>
	<li>图中的红色虚线表示可以配合使用。</li>
</ul><h2 id="Serial"><a name="t16"></a><a name="t16"></a>Serial</h2>

<p><img alt="" height="436" src="https://img-blog.csdnimg.cn/20200220130215487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="815"></p>

<p>垃圾回收的时候，程序是无法执行的。stop-the-world(STW)是停止程序运行，回收线程开始运行，回收结束后程序再接着运行。</p>

<h2 id="Parallel%20Scavenge"><a name="t17"></a><a name="t17"></a>Parallel Scavenge</h2>

<p><img alt="" height="471" src="https://img-blog.csdnimg.cn/20200220130956610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="808"></p>

<p>并行回收，多个线程同时进行垃圾回收。</p>

<h2 id="ParNew"><a name="t18"></a><a name="t18"></a>ParNew</h2>

<p><img alt="" height="474" src="https://img-blog.csdnimg.cn/2020022013122529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="827"></p>

<p>配合CMS的年轻代并行回收。</p>

<h2 id="SerialOld"><a name="t19"></a><a name="t19"></a>SerialOld</h2>

<p>单线程回收算法用于old区域</p>

<h2 id="Parallel%20Old"><a name="t20"></a><a name="t20"></a>Parallel Old</h2>

<p>多线程回收算法用于old区域</p>

<h2 id="CMS"><a name="t21"></a><a name="t21"></a>CMS</h2>

<p>ConcurrentMarkSweep，用于回收老年代，在垃圾回收的同时程序也能运行。(黄色的表示垃圾回收线程，蓝色表示程序执行线程）</p>

<p><img alt="" height="456" src="https://img-blog.csdnimg.cn/20200220132715191.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="835"></p>

<p><img alt="" height="445" src="https://img-blog.csdnimg.cn/20200220133005356.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70" width="836"></p>

<p>调优针对的是Serial、Parallel Scavenge和Serial Old、Parallel New，因为JDK1.8默认的垃圾回收：Parallel Scavenge + Parallel Old。</p>

<h1 id="6%E3%80%81JVM%E5%8F%82%E6%95%B0"><a name="t22"></a><a name="t22"></a>6、JVM参数</h1>

<p>JVM的命令行参数参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html</p>

<p>JVM参数分类：</p>

<ul><li>标准：-开头，所有的HotSpot都支持。 如：java -version</li>
	<li>非标准：-X开头，特定版本HotSpot支持特定命令</li>
	<li>不稳定：-XX开头，下个版本可能取消
	<ol><li>-XX: +PrintFlagsFinal &nbsp; --- 设置值（最终生效值)</li>
		<li>-XX:+PrintFlagsInitial &nbsp;--- 默认值</li>
		<li>-XX:+PrintCommandLineFlags ---命令行参数</li>
	</ol></li>
</ul><h1 id="7%E3%80%81%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE"><a name="t23"></a><a name="t23"></a>7、思维导图</h1>

<p><img alt="" src="https://img-blog.csdnimg.cn/20200220151948590.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzODYxNzM=,size_16,color_FFFFFF,t_70"></p>

<p>参考文档：</p>

<p><a href="https://blogs.oracle.com/jonthecollector/our-collectors" rel="nofollow">https://blogs.oracle.com/jonthecollector/our-collectors</a></p>

<p><a href="https://blogs.oracle.com/jonthecollector/why-not-a-grand-unified-garbage-collector" rel="nofollow">https://blogs.oracle.com/jonthecollector/why-not-a-grand-unified-garbage-collector</a></p>
