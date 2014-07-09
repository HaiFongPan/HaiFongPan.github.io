---
author: leopure
comments: true
date: 2012-08-07 00:49:29+00:00
layout: post
slug: mapreduce-入门笔记
title: MapReduce 入门笔记
wordpress_id: 506
categories:
- MapReduce
- Study Note
tags:
- MapReduce
- 入门
- 笔记
---

7月,严格意义上实习的“First blood”，每天早8晚8，感觉自己都还没学到什么，还没做出点东西来的时候，时间已经不经意间流走，甚至甩都不甩我一下，如今已经是8月头了，再不记录点东西，这个月恐怕一篇博文都写不出了。









MapReduce是实习第一周公司带我的导师要我去了解的一个东西，估计是想看看我的学习能力，理解能力如何，结果我发现自己不是一般的弱，想要变强，只能多看多学多实践。下面记录一些学习到的东西，大部分东西都是摘自论文或者一些博客，还真没什么原创的，不过语言还是自己组织下，加深下印象。（这语气怎么好像跟写技术博一样！明明博客从写流水账\日志就没什么访问量了！）



\n\n







## 1.什么是MapReduce？





《[MapReduce Simplified Data Processing on Large Clusters](http://static.usenix.org/event/osdi04/tech/full_papers/dean/dean.pdf)》这是Google大牛Jeffrey Dean的文章，貌似这抽象模型就是他提出来了。《[我是如何向老婆解释MapReduce的](http://blog.jobbole.com/1321/)》，这篇文章非常简单易懂的把MapReduce的表达了出来，非技术人员也能很快理解MapReduce到底是做什么。MapReduce在我看来，无非就是问题拆分成独立的子问题（Map），然后各个击破、解决，最后把结果（Reduce）（咦！这不就是分治么？）





## 2.MapReduce抽象模型的工作流程\执行方式:





按照网上所有认识MapReduce博文千篇一律的尿性，我要放上一张原理图：





[![](http://www.leopan.me/wp-content/uploads/2012/08/MapReduce执行过程-680x398.jpg)](http://www.leopan.me/wp-content/uploads/2012/08/MapReduce执行过程.jpg)





简言之：在输入数据的记录上进行Map操作，得到中间集<key,value>对集，再对具有相同key的<key,value>对上进行Reduce操作，生成结果输出
  






稍微复杂点，可以根据图中的分布:  








  1. 用户程序将任务分成N片


  2. Master主机想空闲的worker工作机器分配任务Map或者Reduce


  3. 负责Map的机器读取相关的split数据，分析出key\value对缓存到内存中


  4. 将缓存在内存中的key\value对写入本地磁盘，并将数据的相对位置通知master


  5. Master负责告知执行reduce的worker数据的位置，由worker读取数据进行排序操作，将相同key的value合并到一个集合中


  6. 将不同的key和对应的value添加到reduce函数，reduce函数把结果最终写入输出文件


  7. 当所有Map、Reduce任务完成后，通知Master，给用户输出结果





<blockquote>
  
> 
> 更详细版本：直接看Jeffrey Dean的论文吧  

> 
> 
</blockquote>





## 3.MapReduce可应用的场景:





其实在网上搜索MapReduce基本都是关于云计算啊，并行计算等等，看上去很可怕，毕竟云这个东西是新鲜玩意儿，大家都想从这块蛋糕中获取点利益，于是你争我抢的时代开始了，扯远了。





<blockquote>
  
> 
> MR主要可用在  

> 
> 
</blockquote>







  * 分布grep、分布排序、计算URL访问频率、倒转网络连接图、每个主机的术语向量、倒排索引、web访问日志分析、反响索引构建、文档聚类、机器学习，基于统计的机器翻译，生成整个搜索的索引等大规模数据处理；


  * 也能用于数据挖掘、信息提取等


  * mapReduce 适合从事数据密集型、结构化大数据的处理和分析


  * 虽然上面很多技术闻所未闻见所未见，不过记录下来总是好的。





## 4.MapReduce优缺点:





<blockquote>
  
> 
> MapReduce的优点：  

> 
> 
</blockquote>





1.处理大规模数据  

2.隐藏了繁琐的细节：自动并行化、负载均衡、数据分布和灾备管理等  

3.伸缩性好：增加一台服务器，就能将差不多的计算能力接入集群中  






<blockquote>
  
> 
> MapReduce的缺点：  

> 
> 
</blockquote>





1.不适合实时应用的需求  

2.不适合单一的请求  

3.没有索引，只能靠计算能力来处理  






<blockquote>
  
> 
> 这些都是那会儿接触的时候网上各个博文论文摘录过来的
> 
> 
</blockquote>





## 5. MapReduce的task分配机制，出错和冗余机制






    
  * 对于处理map的task，最优的分配是处理本地的数据，因为不必要的网络通信容易带来一定的复杂性，如果本地worker处于运行状态，则应当就近原则选择一个worker执行该map；

    
  * 对于处理reduce任务，选择空闲的worker执行下一条任务列表里面的reduce任务。





出错和冗余机制：






    
  *  如果是对于Worker机产生错误，则在失效的机器上产生的map任务将被重新执行，因为其输出时本地磁盘，所以无法访问，如果是完成的reduce任务，则不会再被执行，因为它的输出在全局文件系统中

    
  * 对于master机产生的错误，则终止计算，检查状态后可重新执行。

    
  * Master如果重复接收到已完成的map任务信息时则会屏蔽掉这类冗余信息以确保结果正确

    
  * Reduce完成后产生的冗余信息，可以通过最后的冗余处理来保证下一次运行的质量。





## 6.《数学之美》中的一个简单MapReduce例子:





最近在拜读吴军大牛的《数学之美》，如果仅仅作为一个想对现有技术算法做一个了解的人来说，书中部分章节极大的满足了这个需求；如果想要对这些技术做一个深度的了解，那在看此书前最好是好好复习复习线性代数、概率论等等。（很明显我还读不懂其中一些比较高深的东西）





书中的例子非常简单：有两个矩阵A(N_N)、B(N_N)，求矩阵C = A*B（A,B过大，一台服务器无法存下），这个时候用MapReduce的原理来解决就非常简单了（其实就是分而治之），首先将矩阵A拆分10个小矩阵，每一个小矩阵有N/10行





[![](http://www.leopan.me/wp-content/uploads/2012/08/QQ截图20120807083748.png)](http://www.leopan.me/wp-content/uploads/2012/08/QQ截图20120807083748.png)





将矩阵B拆分为10个小矩阵，每一个小矩阵有N/10列





[![](http://www.leopan.me/wp-content/uploads/2012/08/QQ截图20120807083757.png)](http://www.leopan.me/wp-content/uploads/2012/08/QQ截图20120807083757.png)





很明显这个时候如果一个服务器处理一个Ai*Bj则能计算出Cij，得到子问题的解，最后将Cij的结果合并起来便是所求矩阵C了，一台服务器搞不定的问题，多台服务器搞定了





有人肯定会说这个问题简直是太水了，图森破，但是正是这个简单的例子体现MapReduce的根本原理，往往我们解决大问题的灵感都来自于这些最常见，最简单的问题。





## 7.总结：





文章水到这里也水得差不多了，东平西凑，没什么自己的见解，权当留作学习回顾之用，想要更多的了解MR还可以去了解Hadoop这类怪兽级框架。PS:真正想要学好一样东西，关键还是看兴趣。





<全文完>



