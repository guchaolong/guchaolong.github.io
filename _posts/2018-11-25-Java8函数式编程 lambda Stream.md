---
layout:     post
title:      Java8、Java9
subtitle:   开发积累
date:       2018-11-22
author:     guchaolong
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
       - Java
---
>函数式编程  


# 函数式编程、lambda、 Stream Api
- ## 函数式编程  
和面向对象编程、面向过程编程类似，函数式编程是另一种编程范式、思想、方法论  
在函数式编程中，函数是第一等公民，可以被赋值，可以被当作参数传递

- ## 匿名类和函数接口
``` 
 Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("hello");
            }
        };
```
- __匿名类__：  
以上是匿名类的典型，首先接口是不能实例化的，接口也没有构造方法，被new修饰是因为，new的时候实际是创建了一个匿名的实现了Runable接口的
实现类，实现了run方法，如果没有匿名类，就需要单独去创建一个类实现run方法，代码就会更加繁琐，于是就有了lambda表达式  

- __函数接口__：  
java8之后接口里可以有默认方法，有且仅有一个抽象方法的接口，叫做函数接口，函数接口可以用lambda表达式  
java8以前的函数接口有Runnable Callable Comparator等
Java8以后，增加了 java.util.function下的一些类 如Predicate<T>、Supplier<T>等  

-  ## Lambda表达式
（）->{};  参数列表->语句

- ## Stream API
__流__ : 流水线，一步一步处理数据，最终得到想要的数据  
  
__并行流__ : 有多个数据，如果是单核CPU,就是在一条流水线上处理数据，处理完一个再处理另一个；如果是在多核CPU上，每个CPU上可以处理一个数据
，每个CPU都相当于一条流水线，同时进行  

__中间操作__:  返回一个流  

__终止操作__:不返回流  

__惰性求值__:如果流没有做中止操作，中间操作就不会执行  


   
__map()__  转换，例如源数据是一个字符串，可以用map获得该字符串的长度  

__flatMap()__  A对象，A对象下面有个集合类型的属性b,使用map()，每个A都会得到一个b的列表，使用flatMap(),可以得到所有A对象的所有b属性的一个列表  

__filter()__  过滤  

__peek()__  peek接收一个没有返回值的λ表达式，可以做一些输出，外部处理等。map接收一个有返回值的λ表达式，之后Stream的泛型类型将转换为map参数λ表达式返回的类型

__reduce()__  对所有数据进行处理得到一个结果  如下 输出Optional\[my|name|is|faker]  13 0是初始值
```aidl
Optional<String> reduce = Stream.of(str.split(" ")).reduce((s1, s2) -> s1 + "|" + s2);
System.out.println(reduce);

Integer reduce1 = Stream.of(str.split(" ")).map(s -> s.length()).reduce(0, (L1, L2) -> L1 + L2);
System.out.println(reduce1);
```  

__Stream操作机制__ :1.所有操作都是链式调用，如 要迭代A B C三个元素，操作有a b c三个，对A执行a b c，然后对B执行a b c,不会对A执行a后就对B执行a...  


# JDK9的Reactive Stream
Jdk9的引入的一套标准，是基于发布订阅模式数据处理的规范，和Jdk8的Stream没关系，代码也没有耦合  
在JDK9里面其实叫Flow API

- ## 背压  
发布者和订阅者之间的互动，订阅者可以告诉发布者 需要多少数据，起到一个调节数据流量的作用  
例子：自来水公司（发布者）、家庭（订阅者） 被动接受，有了背压（水龙头），需要就打开，不需要就关闭

- ## 结构  
  Flow  
  Publisher :发布者  
  Subscriber :订阅者  
  Subscription :订阅关系  
  Processor：对数据进行过滤和加工   
  能实现背压的原因 订阅者会把收到的数据先放到缓冲池（默认256条数据），发布者的submit()方法是一个阻塞方法，当订阅者的缓冲池满了，就不会再发布  
  订阅者消费了一条数据之后，发布者又再发布一条
  
# 响应式编程
1. 同步、异步：  
    同步就是一件事做完之后，再做另一件事；异步就是做一件事，还没完成，在等待期间开始做另一件事
2. 同步Servlet、异步Servlet:  
    前者是阻塞的，tomcat容器管理Servlet线程(线程调用servlet中的service方法)，一个请求对应一个Servlet线程，一个请求到达之后，tomcat容器启动一个Servlet线程去处理，处理完这个请求之后，这个Servlet线程才会去处理其他请求  
    后者是非阻塞的，一个请求到达，启动一个Servlet线程处理，在等待业务逻辑处理的时候，这个Servlet线程可以去处理其他请求，这样就可以提高并发量 
3. Reactor = jdk8的Stream + jdk9的Reactive Stream  
    Mono和Flux都实现了Publisher  
    Mono 0-1个元素  
    Flux 0-n 个元素
    

 

