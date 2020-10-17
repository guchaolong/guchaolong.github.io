---
layout:     post
title:      IO NIO AIO Netty
subtitle:   
date:       2019-05-31
author:     guchaolong
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
category: 操作系统
tags:
       - Linux
---
> IO NIO AIO Netty

# 相关文章

[](https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453141327&idx=2&sn=0e96562ddc4896b19653bbed77002154&scene=19#wechat_redirect)

[](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484235&idx=1&sn=4c3b6d13335245d4de1864672ea96256&chksm=ebd7424adca0cb5cb26eb51bca6542ab816388cf245d071b74891dd3f598ccd825f8611ca20c&scene=21#wechat_redirect)



# kernel

> 操作系统内核
>
> 1. 向下**管理硬件**
> 2. 向上提供**系统调用**System call (sc）

机器启动，内核也是一个程序，开机时，内核要加载到内存，其他应用程序也加载到内存，然后CPU就可以执行

加载到内存后，内核所占的空间：**内核空间**，剩余的应用程序所占的空间：**用户空间**

**保护模式**：CPU执行内核的时候，可以访问到整个内存，但CPU执行用户空间时不能访问内核空间的，减少对系统的破坏

**中断**：内核会提供一些对底层操作的封装，给用户程序使用，即给程序提供**系统调用**，程序必然是要使用硬盘这些硬件的，要访问内核，但是有保护模式，怎么办？**中断**，如**时钟中断**，CPU停止执行当前的程序，保护现场，去执行其他的程序

当程序对内核进行系统调用的时候，就会产生中断，调CPU的 INT指令，产生**用户态和内核态的切换**，

切换要保护现场 恢复现场 都是比较消耗性能的



# 同步/异步 阻塞/非阻塞

同步异步：关注的是消息通信机制，做一件事，这件事完成后是否还要我自己去处理，是，则同步；开始做事之前，我先写好完成后要怎么处理，如果做完事后，直接调预先的处理方法，自己不用去关注了

阻塞/非阻塞，关注的是**能不能动**，（用户线程是不是要**等待**，会不会立即返回，如果没返回，就要等待）



# **I/O**

应用程序都是运行在**用户空间**的，所以它们能操作的数据也都在用户空间，

I/O就是 数据从**内核空间**到**用户空间** 的相互拷贝



# BIO



应用程序必然是要和内核交互的，对内核进行系统调用，

对于服务端，要新建一个Socket，绑定端口

> 对应到内核中有:
>
> socket() 返回一个文件描述符 比如3、
>
> bind(3，8090)绑定端口、
>
> listen()开启监听
>
> ...
>
> accept(3,			（会阻塞）
>
> recv(5,					(也会阻塞)



```java

/**
 * BIO
 *
 * 阻塞IO
 * accept() 和 readLine（）方法，如果没有数据是不会返回值的，会一直等待
 */
public class ServerSocketBIO {
    public static void main(String[] args) {

        try {
            //服务端开启ServerSocket之后，kernel会开启一个socket，绑定端口，并进行监听
            ServerSocket serverSocket = new ServerSocket(8090);
            System.out.println("Tcp server ready");


            while (true) {
                //阻塞1，等待客户端来连接,如果没有客户端连接过来，则一直等待在这，下面的代码不会执行
                //也就是说如果没有accept到，就不会返回这个client
                Socket client = serverSocket.accept();

                System.out.println(client.getPort());

                new Thread(() -> {
                    try {
                        BufferedReader reader = new BufferedReader(new InputStreamReader(client.getInputStream(), StandardCharsets.UTF_8));
                        BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(client.getOutputStream(), StandardCharsets.UTF_8));

                        //阻塞2，等待输入，如果有客户端连接进来了，但是没有传过来数据，也会一直等待在这
                        //也就是说如果没有读取到，就不会返回这个cmd
                        String cmd = reader.readLine();

                        if ("time".equals(cmd)) {
                            writer.write(LocalDateTime.now().toString() + "\n");
                            writer.flush();
                        } else {
                            writer.write("sorry?\n");
                            writer.flush();
                        }

                        client.close();
                        serverSocket.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }).start();


            }


        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

以上代码，处于阻塞2处的时候，有客户端连接过来后，服务端是accept不到的，因为它还在**等待**上一个的输入

因此，传统的最古老的网络编程是：**每连接对应每线程**

循环，accept一个连接之后，创建一个线程，在自己的线程中read，读取的时候阻塞也是发生在它自己的现场中

问题：如果有10000个连接来了，那我就要创建10000个线程去处理，很消耗资源



# NIO

> 因为是进行系统调用，而内核对应的调用是阻塞的，所以是否阻塞，受制于内核
>
> 所以需要内核提供非阻塞的系统调用

new io是指java.nio包下提供的一些新的接口，必如ByteBuffer，Channel，Selector， SocketChannel， ServerSocketChannel，比如，可以通过ServerSocketChannel.open()开启一个服务端socket，并绑定端口，最主要的是可以配置是否阻塞，设置非阻塞之后其实就对应到了操作系统内核中的NONBLOCKING，非阻塞，就是调方法不会停住，绝对会有返回，可以根据返回值判断，是否有连接进来，或者或者是否有数据可读，我的方法是不会停下的，传统的BIO的话，我需要对每个连接都新建一个线程去读取对应连接的数据，因为BIO读取的时候也是会阻塞的，NIO的话，一个线程就可以把所有的活都干了，我只要不断的轮询，内核每次都会给我返回，弊端是每次都询问内核，如果有10000个连接，只有1个发了数据，那9999次都是无意义的

```java
/**
 * 非阻塞IO
 * 服务端：ServerSocketChannel
 * 客户端：SocketChannel
 * <p>
 * 它的accept()和read()方法是会立即返回的，不需要等待，可以根据这个返回结果做相应的处理，
 * 但是需要不断的轮询，也就是死循环调这两个方法，轮询内核
 */
public class ServerSocketNIO {

    public static void main(String[] args) throws Exception {


        List<SocketChannel> clients = new LinkedList<>();
        ServerSocketChannel ss = ServerSocketChannel.open();
        ss.bind(new InetSocketAddress(9090));

        //重点  对应到内核 NONBLOCKING
        ss.configureBlocking(false);

        //整个主线程一直在轮询，但是这里是用户线程主动去轮询内核
        while (true) {
            Thread.sleep(1000);

            //这里不会阻塞，因为会返回一个结果，后续做相应处理,
            //调用内核了
            //1. 如果没有客户端连进来 返回值？ BIO的时候会一直等着，不会有返回，配置了Nonblocking, NIO,会返回-1（或者null）
            //2. 有客户端连进来，就能接受的客户端，对应到内核中就是一个fd (文件描述符）
            SocketChannel client = ss.accept();

            if (client == null) {
                System.out.println("null...");
            } else {
                client.configureBlocking(false);
                int port = client.socket().getPort();
                System.out.println("client port: " + port);
                clients.add(client);
            }

            /*
            分配缓冲区 （这个缓冲区可以在堆里，也可以在堆外）
             */
            ByteBuffer buffer = ByteBuffer.allocateDirect(4096);

            for (SocketChannel c : clients) {//串行化
                int num = c.read(buffer);// >0 -1 0  能得到一个结果，后续做相应的处理，所以这里也不会阻塞
                if (num > 0) {
                    buffer.flip();
                    byte[] aaa = new byte[buffer.limit()];
                    buffer.get(aaa);

                    String b = new String(aaa);
                    System.out.println(c.socket().getPort() + " : " + b);
                    buffer.clear();
                }
            }
        }
    }
}
```



# 多路复用IO

> 由于以上NIO的弊端，内核进一步升级，提供了多路复用器（select、poll、epoll)
>
> select和poll 是一类
>
> epoll

int select(int nfds, fd_set *readfds, fd_set *writefds,
                  fd_set *exceptfds, struct timeval *timeout);

允许程序监控多个文件描述符，



**如果是程序自己读取IO,那么这个IO模型，无论是BIO,NIO,多路复用器，它都是同步IO模型**，多路复用器只是会给你返回一个状态，具体读写还是你要自己去做，只不过在同步里面有阻塞和非阻塞



优势：

通过一次系统调用，把fds传递给内核，内核自己进行遍历，减少了系统调用的次数

弊端：

1. 传递重复的fds,解决方案，内核开辟空间保留fd
2. 每次select poll都要重新遍历全量的fd

```java

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

/**
 * Description: 单线程版多路复用
 *
 * @author AA
 * @date 2020/10/13 17:15
 */

/**
 * 单线程版非阻塞IO：需要用户线程主动轮询内核，是否有数据
 * 在Channel的基础上，有引入了Selector(多路复用器）
 * Channel可以把自己注册到Selector上，把这个轮询的过程交给内核的selector去做，我们循环的轮询Selector就行
 * selector.select（）可以知道是否有事件到达
 */
public class SocketMultiplexingSingleThreadV1 {

    /*
    NIO

    nonblocking: socket网络，内核机制
    new  : jdk {channel, buffer, select(多路复用器）}

     */

    private ServerSocketChannel server = null;
    private Selector selector = null;

    int port = 9090;

    public static void main(String[] args) {
        SocketMultiplexingSingleThreadV1 service = new SocketMultiplexingSingleThreadV1();
        service.start();
    }

    public void initServer() {
        try {
            server = ServerSocketChannel.open();
            server.configureBlocking(false);
            server.bind(new InetSocketAddress(port));

            selector = Selector.open();


            /*
            FIXME 我的理解： 把serverChannel的可ACCEPT事件注册到多路复用器上

            这样的就不用主动轮询内核是否有连接可以accept，
            把这个事情交给多路复用器去完成
             */
            server.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    public void start() {

        initServer();
        System.out.println("服务器启动了..................");

        //死循环
        while (true) {
            try {
                //问下内核，是否有事件到达
                while (selector.select(0) > 0) {

                    //从多路复用器，取出有效的key
                    Set<SelectionKey> selectionKeys = selector.selectedKeys();
                    Iterator<SelectionKey> iter = selectionKeys.iterator();
                    while (iter.hasNext()) {
                        SelectionKey key = iter.next();
                        iter.remove();

                        //是否可以建立连接
                        if (key.isAcceptable()) {
                            acceptHandler(key);
                        } else if (key.isReadable()) {//是否可以读数据
                            System.out.println("一般由用户数据到达会触发，但啥时候会疯狂触发呢");
                            readHandler(key);
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }


        }
    }


    public void acceptHandler(SelectionKey key) {
        try {
            ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
            SocketChannel client = ssc.accept();
            client.configureBlocking(false);
            //针对每个客户端都新建一个buffer
            ByteBuffer buffer = ByteBuffer.allocate(8192);

            //把客户端channel，的可读事件，连同buffer,注册到多路复用器,如果客户端写入数据，那server就可以来读取了
            client.register(selector, SelectionKey.OP_READ, buffer);

            System.out.println("==========================================================================================");
            System.out.println("新客户端：" + client.getRemoteAddress());
            System.out.println("此时的select：" + selector);
            System.out.println("此时的ssc：" + ssc);
            System.out.println("此时的client：" + client);
            System.out.println("此时的buffer：" + buffer);
            System.out.println("==========================================================================================");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void readHandler(SelectionKey key) {
        SocketChannel client = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer) key.attachment();

        //先清空buffer
        buffer.clear();

        int read = 0;
        try {
            while (true) {
                //把客户端里的数据读到buffer里
                read = client.read(buffer);
                if (read > 0) {
                    buffer.flip();
                    while (buffer.hasRemaining()) {
                        client.write(buffer);
                    }
                    buffer.clear();
                } else if (read == 0) {
                    break;
                } else {//-1  客户端出现close_wait  bug  死循环  CPU100%
                    client.close();
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


```



> 多线程版多路复用



```java

/**
 * Description: 多线程版多路复用
 *
 * @author AA
 * @date 2020/10/13 17:15
 */

/**
 * 多线程版非阻塞IO多路复用
 * <p>
 * 开启3个线程,每个线程维护一个自己的Selector, 创建一个队列类型的数组，让3个线程共享这个队列
 * 线程1 boss，只负责处理isAcceptable（）的事件，然后把它放到队列里面
 * 线程2 和 线程3是两个worker, 负则处理读， 到队列中各自的位置上取出数据做处理
 */
public class SocketMultiplexingThreadV2 {

    /*
    NIO

    nonblocking: socket网络，内核机制
    new  : jdk {channel, buffer, select(多路复用器）}

     */

    private ServerSocketChannel server = null;
    private Selector selector1 = null;
    private Selector selector2 = null;
    private Selector selector3 = null;
    int port = 9090;


    public static void main(String[] args) {
        SocketMultiplexingThreadV2 service = new SocketMultiplexingThreadV2();
        service.initServer();

        NioThread t1 = new NioThread(service.selector1, 2);
        NioThread t2 = new NioThread(service.selector2);
        NioThread t3 = new NioThread(service.selector3);


        t1.start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        t2.start();
        t3.start();

        System.out.println("服务器启动了。。。");

        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    public void initServer() {
        try {
            server = ServerSocketChannel.open();
            server.configureBlocking(false);
            server.bind(new InetSocketAddress(port));

            selector1 = Selector.open();
            selector2 = Selector.open();
            selector3 = Selector.open();


            /*
            FIXME 我的理解： 把serverChannel的可ACCEPT事件注册到多路复用器上

            这样的就不用主动轮询内核是否有连接可以accept，
            把这个事情交给多路复用器去完成
             */
            //只在selector1上注册
            server.register(selector1, SelectionKey.OP_ACCEPT);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


class NioThread extends Thread {
    Selector selector = null;
    static int selectors = 0; //worker的数量
    int id = 0;
    boolean isBoss = false;

    //注意，是static的，所以boss workerd都能访问到
    static BlockingQueue<SocketChannel>[] queue;
    static AtomicInteger idx = new AtomicInteger();

    //给Boss用的构造器
    NioThread(Selector selector, int n) {
        this.selector = selector;
        selectors = n;
        isBoss = true;

        queue = new LinkedBlockingQueue[selectors];

        for (int i = 0; i < n; i++) {
            queue[i] = new LinkedBlockingQueue<>();
        }
        System.out.println("Boss启动");
    }

    //给worker用到构造器
    NioThread(Selector sel) {
        this.selector = sel;
        id = idx.getAndIncrement() % selectors;
        System.out.println("worker " + id + "启动");
    }

    @Override
    public void run() {
        try {
            while (true) {
                while (selector.select(10) > 0) {
                    Set<SelectionKey> selectionKeys = selector.selectedKeys();
                    Iterator<SelectionKey> iterator = selectionKeys.iterator();
                    while (iterator.hasNext()) {
                        SelectionKey key = iterator.next();
                        iterator.remove();
                        if (key.isAcceptable()) {
                            acceptHandler(key);
                        } else if (key.isReadable()) {
                            readHandler(key);
                        }
                    }
                }

                //boss不参与  只有worker根据分配，分别注册自己的client
                if (!isBoss && !queue[id].isEmpty()) {
                    ByteBuffer buffer = ByteBuffer.allocate(8192);
                    SocketChannel client = queue[id].take();
                    client.register(selector, SelectionKey.OP_READ, buffer);
                    System.out.println("===================================================");
                    System.out.println("新客户端：" + client.socket().getPort() + "分配到worker: " + id);
                    System.out.println("===================================================");
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void acceptHandler(SelectionKey key) {
        try {
            ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
            SocketChannel client = ssc.accept();
            client.configureBlocking(false);

            int num = idx.getAndIncrement() % selectors;//0 1
            queue[num].add(client);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void readHandler(SelectionKey key) {
        SocketChannel client = (SocketChannel) key.channel();
        ByteBuffer buffer = (ByteBuffer) key.attachment();

        //先清空buffer
        buffer.clear();

        int read = 0;
        try {
            while (true) {
                //把客户端里的数据读到buffer里
                read = client.read(buffer);
                if (read > 0) {
                    buffer.flip();
                    while (buffer.hasRemaining()) {
                        client.write(buffer);
                    }
                    buffer.clear();
                } else if (read == 0) {
                    break;
                } else {//-1  客户端出现close_wait  bug  死循环  CPU100%
                    client.close();
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```



# select\poll  和 epoll

select\poll他们在内核没有空间，而是在jvm中

epoll是在内核中开辟空间



# Netty

## 介绍



## 线程模型

1. 传统的阻塞I/O服务模型
2. Reactor模式
   1. 单Reactor单线程
   2. 单Reactor多线程
   3. 主从Reactor多线程 （**netty采用**）



## Netty 核心组件

1. **Bootstrap、ServerBootstrap**

   > 一个 Netty 应用通常由一个 Bootstrap 开始，主要作用是**配置**整个 Netty 程序，**串联各个组件**，Netty 中 ```Bootstrap``` 类是客户端程序的**启动引导类**，```ServerBootstrap``` 是服务端**启动引导类**

2. **Future、ChannelFuture**

   > 在 Netty 中所有的 IO 操作都是**异步**的，不会**立刻知道**某个事件是否完成处理。
   >
   > 但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 ```Future``` 和 ```ChannelFutures```，用来注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件

3. **Channel**

   > Netty 网络通信的组件，能够用于执行网络 I/O 操作

4. **Selector**

   > Netty 基于 ```Selector``` 对象实现 **I/O 多路复用**，通过 Selector一个线程可以监听多个连接的 ```Channel``` 事件。
   >
   > 
   >
   > 当向一个 Selector 中注册 Channel 后，**Selector 内部的机制**就可以**自动不断地查询**（Select）这些注册的 Channel 是否有已就绪的 I/O 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地管理多个 Channel 

5. **NioEventLoop**

   > ```NioEventLoop``` 中维护了**一个线程**和**任务队列**，支持异步提交执行任务，线程启动时会调用 NioEventLoop 的 run 方法，执行 I/O 任务和非 I/O 任务
   >
   > 
   >
   > 1. I/O 任务，即 ```selectionKey ```中 ```ready``` 的事件，如 ```accept、connect、read、write``` 等，由 ```processSelectedKeys``` 方法触发
   > 2. 非 IO 任务，添加到 ```taskQueue``` 中的任务，如 ```register0、bind0``` 等任务，由 ```runAllTasks``` 方法触发
   >
   > 两种任务的执行时间比由变量 ```ioRatio``` 控制，默认为 50，则表示允许非 IO 任务执行的时间与 IO 任务的执行时间相等

6. **NioEventLoopGroup**

   > ```NioEventLoopGroup```，主要管理 eventLoop 的生命周期，可以理解为一个线程池，内部维护了一组线程，每个线程（```NioEventLoop```）负责处理多个 ```Channel ```上的事件，而一个``` Channel``` 只对应于一个线程

7. **ChannelHandler**

   > ```ChannelHandler``` 是一个接口，处理 I/O 事件或拦截 I/O 操作，并将其**转发**到其 ```ChannelPipeline```（**业务处理链**）中的下一个处理程序
   >
   > 
   >
   > ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类：
   >
   > - ```ChannelInboundHandler``` 用于处理入站 I/O 事件。
   > - ```ChannelOutboundHandler``` 用于处理出站 I/O 操作。
   >
   > 
   >
   > 或者使用以下适配器类：
   >
   > - ```ChannelInboundHandlerAdapter``` 用于处理入站 I/O 事件。
   > - ```ChannelOutboundHandlerAdapter``` 用于处理出站 I/O 操作。
   > - ```ChannelDuplexHandler``` 用于处理入站和出站事件。

8. **ChannelHandlerContext**

   > 保存 ```Channel``` 相关的所有**上下文信息**，同时关联一个 ```ChannelHandler``` 对象

9. **ChannelPipline**

   > 保存```ChannelHandlerContext``` 的 **双向链表**，```ChannelHandlerContext``` 中关联着一个```ChannelHandler```用于处理或拦截 ```Channel``` 的入站事件和出站操作。
   >
   > 
   >
   > 它实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel 中各个的 ChannelHandler 如何相互交互。
   >
   > 
   >
   > 在 Netty 中**每个 ```Channel``` 都有且仅有一个 ```ChannelPipeline``` 与之对应**。
   >
   > 
   >
   > 关于 Netty 的简介就先说这么多，后面会带着 Socket 通信应该解决的问题和上面提到的 Netty 关键组件讲解 Netty 是如何实现高性能网络通信的

   入站事件和出站事件在同一个链表中，入站事件从链表head往后传递到最后一个入站的handler，出站事件从链表tail往前传递到最前一个出站的handler，两种类型的handler互不干扰