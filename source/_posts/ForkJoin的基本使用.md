---
title: ForkJoin的基本使用
date: 2019-11-07 15:44:29
tags: ['并发编程','性能优化']
---

***简介：***“分而治之” 按照设定的阈值进行分解成多个计算，然后将各个计算结果进行汇总。相应的ForkJoin将复杂的计算当做一个任务。而分解的多个计算则是当做一个子任务。将大的任务缩小化，forkjoin是你成为架构师的必经之路。

<!--more-->

## 1.工作窃取算法 

fork/join优秀的地方就在于这个算法,假如我们需要做一个比较大的任务，我们可以把这个任务分割为若干互不依赖的子任务，把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一 一对应，比如A线程负责处理A队列里的任务。但是有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。而在这时它们会访问同一个队列，所以为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。

![](http://dsvip1.vip/forkjoin2.png)

想到真正意义的理解，更多的还是要查看源码。

![](http://dsvip1.vip/forkjoin1.png)

## 2.fork/join的基本使用

##### (1)ForkjoinTask

 是任务本身,使用它来创建任务,提供fork(),join(),compute()等核心方法,提供两个实现子类: 

 RecursiveTask :需要返回值时;
 RecursiveAction:不需要返回值时; 

##### (2) ForkJoinPool:

  ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务. 

##### (3)基本使用实例

如有很多的数据需要往数据库中存储，下面使用forkjoin来实现对其操作

```java
package com.dangle.MyThread;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;

/**
 * @author by danglea on 2019/11/7
 */
public class MyForkJoin extends RecursiveTask<Integer> {
    List<Integer> list; //模拟数据库数据
    public  MyForkJoin(List<Integer> list){   //构造方法
        this.list=list;
    }

    @Override
    protected Integer compute() {  //重写RecursiveTask的compute方法
        if(list.size()<3){
            System.out.print("小于3");
            return computeDirectly();
        }else{
            int size=list.size();
            MyForkJoin my1=new MyForkJoin(list.subList(0,size/2)); //分而治之思想
            MyForkJoin my2=new MyForkJoin(list.subList(size/2,list.size()));
            //
            invokeAll(my1,my2); 
            /*my1.fork();分别执行子任务
            my2.fork();*/
            return my1.join()+my2.join();
        }
    }

    private Integer computeDirectly() {
        try {

            Thread.sleep(1000);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("插入了"+ Arrays.toString(list.toArray())+list.size());
        return list.size();
    }

    public static void main(String[] args) throws  Exception {
        ForkJoinPool forkJoinPool = new ForkJoinPool(8);
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(1);
        list.add(1);
        list.add(1);
        list.add(1);
        list.add(1);
        list.add(1);
        list.add(1);
        list.add(1);
        list.add(1);
        list.add(1);
        list.add(1);

        MyForkJoin batchInsertTask = new MyForkJoin(list);
        long t1 = System.currentTimeMillis();
        ForkJoinTask<Integer> reslut = forkJoinPool.submit(batchInsertTask);
        System.out.println(reslut.get());
        long t2 = System.currentTimeMillis();
        System.out.println(t2-t1);

    }
}

```

invokeAll()和fork()的区别，为什么fork的执行时间长呢？

线程池的线程是有数，比如现在有400的任务，有四个工人，1，2，3，4使用fork方法，就使用将400的任务分给1，2，每个200，但是1，2不干活只是分配任务，分给3，4.但是3，4也是继续向下分分给其他的一直分到100，所以执行结束就需要7个工人在一天的情况下，而invokeAll方法则是直接1，2，3，4.完成·任务线程的充分利用，而不存在等待的线程。

