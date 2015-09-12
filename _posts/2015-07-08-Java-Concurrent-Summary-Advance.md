---
layout: post
title:  Java Concurrent Summary Advance
date:   2015-07-08 17:43:19
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---
### 并发容器和框架  
<br>
- 如何让一段程序并发的执行，并最终汇总结果？   
拆分计算，使用Callable对象封装每个计算任务，通过Executor执行，在Future的get中获取计算结果。最后在主线程中汇总计算结果。

- 如何合理的配置java线程池？如CPU密集型的任务，基本线程池应该配置多大？IO密集型的任务，基本线程池应该配置多大？用有界队列好还是无界队列好？任务非常多的时候，使用什么阻塞队列能获取最好的吞吐量？    
根据任务的性质（比如IO密集，或者CPU密集），配置核心线程的数量，最大线程数，选择好合适的队列，以及拒绝策略。

