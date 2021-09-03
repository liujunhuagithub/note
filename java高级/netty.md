# 特点

Netty是由JBOSS提供的一个Java开源框架，现为Github上的独立项目。
Netty是一个**异步的、基于事件驱动**的**网络**应用框架，用以快速开发高性能、高可靠性的网络IО程序。
Netty主要针对在**TCP**协议下，面向**Clients端**的高并发应用，或者Peer-to-Peer场景下的大量数据持续传输的应用。
Netty本质是一个Java NIO**客户端/服务器**框架，适用于服务器通讯相关的多种应用场景
要透彻理解Netty ,需要先学习NIO，这样我们才能阅读Netty 的源码。



Netty线程模式(Netty主要基于主从Reactor多线程模型做了一定的改进，其中主从Reactor多线程模型有多个Reactor)



![image-20210902175413233](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210902175413233.png)

# IO模型

## 结构

# 问题解决

## 黏包





![image-20210902165050046](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210902165050046.png)

![image-20210902165146139](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210902165146139.png)

黏包、半包：OS发送网络时缓存批量发送导致边界问题

解决断线

零拷贝：针对transerTo，整个过程仅只发生了一次用户态与内核态的切换，数据拷贝了2次。所谓的【零拷贝】，并不是真正无拷贝，而是在不会拷贝重复数据到 jvm内存中，零拷贝的优点有
·更少的用户态与内核态的切换
·不利用cpu计算，减少cpu缓存伪共享·零拷贝适合小文件传输

OS支持的零拷贝有 mmap(内存映射)和sendFile。那么，他们在OS里，到底是怎么样的一个的设计?我们分析 mmap 和sendFile 这两个零拷贝

mmap通过内存映射，将文件映射到内核缓冲区，同时，用户空间可以共享内核空间的数据。这样，在进行网络传输时，就可以减少内核空间到用户控件的拷贝次数。

sendFile优化。Linux 2.1版本提供了
sendFile函数，其基本原理如下:数据根本不经过用户态，直接从内核缓冲区进入到SocketBuffer，同时，由于和用户态完全无关，就减少了一次上下文切换Linux在2.4版本中，做了一些修改，避免了从内核缓冲区拷贝到 Socket
buffer的操作，直接拷贝到协议栈，从而再一次减少了数据拷贝。具体如下图和小结:

![image-20210902175231190](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210902175231190.png)



![image-20210902173838520](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210902173838520.png)

# Reactor模型

Reactor 的数量和处理资源池线程的数量

Reactor: Reactor在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理程序来对I0事件做出反应。

Handlers:处理程序执行I/O事件要完成的实际事件。Reactor通过调度适当的处理程序来响应/0事件，处理程序执行非阻塞操作。



建立连接accept事件和读写事件分别处理





## 传统阻塞IO

采用阻塞IO模式获取输入的数据
每个连接都需要独立的线程完成数据的输入、业务处理、数据输出

![image-20210903084833502](image-20210903084833502.png)



基于线程池复用线程资源:不必再为每个连接创建线程，将**连接完成后**的业务处理任务分配给线程进行处理，一个线程可以处理多个连接的业务。





## 单Reactor单线程

基于I/O复用模型:多个连接共用一个阻塞对象，应用程序只需要在一个阻塞对象等待，无需阻塞等待所有连接。当某个连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理

![image-20210903085449979](image-20210903085449979.png)

Select是前面I/0复用模型介绍的标准网络编程API，可以实现应用程序通过一个阻塞对象监听多路连接请求
Reactor对象通过Select监控客户端请求事件，收到事件后通过Dispatch进行分发如果是建立连接请求事件，则由 Acceptor通过Accept处理连接请求，然后创建一个Handler对象处理连接完成后的后续业务处理
如果不是建立连接事件，则 Reactor会分发调用连接对应的 Handler来响应Handler会完成 Read→>业务处理→Send的完整业务流程



优点:模型简单，没有多线程、进程通信、竞争的问题，全部都在一个线程中完成缺点:性能问题，只有一个线程，无法完全发挥多核CPU的性能。Handler在处理某个连接上的业务时，整个进程无法处理其他连接事件，很容易导致性能瓶颈
缺点:可靠性问题，线程意外终止，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障
使用场景:客户端的数量有限，业务处理非常快速，比如Redis在业务处理的时间复杂度O(1)的情况

优点:可以充分的利用多核cpu的处理能力
缺点:多线程数据共享和访问比较复杂，reactor处理所有的事件的监听和响应，在单线程运行,在高并发场景容易出现性能瓶颈.



![image-20210903090835667](3090835667.png)



## 单Reactor多线程

![image-20210903085927658](image-20210903085927658.png)

![image-20210903085313212](image-20210903085313212.png)



Reactor对象通过select监控客户端请求事件,收到事件后，通过dispatch进行分发如果建立连接请求,则右Acceptor通过
accept处理连接请求,然后创建一个Handler对象处理完成连接后的各种事件
如果不是连接请求，则由reactor分发调用连接对应的handler来处理
handler只负责响应事件,不做具体的业务处理,通过read读取数据后，会分发给后面的worker线程池的某个线程处理业务
worker线程池会分配独立线程完成真正的业务,并将结果返回给handler
handler收到响应后，通过send将结果返回给client





![image-20210903090937499](image-20210903090937499.png)

## 主从 Reactor多线程

![image-20210903090336533](image-20210903090336533.png)

![image-20210903091014081](081.png)

3，第三种模型比起第二种模型，是将Reactor分成两部分，mainReactor负责监听server socket，accept新连接，并将建立的socket分派给subReactor。subReactor负责多路分离已连接的socket，读写网·络数据，对业务处理功能，其扔给worker线程池完成。通常，subReactor个数上可与CPU个数等同。



subReacror可有多个

Reactor主线程MainReactor对象通过select监听连接事件,收到事件后,通过Acceptor处理连接事件
当Acceptor处理连接事件后,MainReactor将连接分配给SubReactor
subreactor将连接加入到连接队列进行监听并创建handler进行各种事件处理
当有新事件发生时，subreactor就会调用对应的handler处理handler通过read 读取数据,分发给后面的worker线程处理worker 线程池分配独立的worker线程进行业务处理,并返回结果
handler收到响应的结果后,再通过send将结果返回给clientReactor主线程可以对应多个Reactor子线程

1)优点:父线程与子线程的数据交互简单职责明确，父线程只需要接收新连接，子线
程完成后续的业务处理。
2)优点:父线程与子线程的数据交互简单，Reactor主线程只需要把新连接传给子线
程，子线程无需返回数据。
3)缺点:编程复杂度较高
结合实例:这种模型在许多项目中广泛使用，包括 Nginx主从Reactor多进程模型，Memcached主从多线程，Netty主从多线程模型的支持

响应快，不必为单个同步时间所阻塞，虽然Reactor本身依然是同步的可以最大程度的避免复杂的多线程及同步问题，并且避免了多线程/进程的切换开销
扩展性好，可以方便的通过增加Reactor实例个数来充分利用CPU资源复用性好，Reactor模型本身与具体事件处理逻辑无关，具有很高的复用性

![查看源图像](ghfsrdgsfdgfdsfd.png)



# Netyy结构

## 交互流程

![img](a24e12f2282106f1accab91b88e2a5e3.png)

## netty组件

### Channel

 Channel是 Java NIO 传输数据的通道，可以被打开或关闭，连接或者断开连接

### EventLoop 与 EventLoopGroup

#### EventLoop 

 EventLoop 定义了Netty的核心抽象，用来处理连接的生命周期中所发生的事件

NioEventLoop内部采用串行化设计，从消息的读取->解码->处理->编码->发送，始终由I0线程NioEventLoop负责

NioEventLoop表示一个**不断循环的执行处理任务的线程**，**每个NloEventLoop都有一个selector,**用于监听绑定在其上的socket的网络通讯`Channel`。EventLoop 与线程的关系是 1:1

 Netty 为每个 Channel **固定分配**了一个 EventLoop（Channel整个生命周期不可修改绑定），用于处理用户连接请求、对用户请求的处理等所有事件。EventLoop 本身只是一个线程驱动，在其生命周期内只会绑定一个线程，让该线程处理一个 Channel 的所有 IO 事件。

 一个 Channel 一旦与一个 EventLoop 相绑定，那么在 Channel 的整个生命周期内是不能改变的。一个 EventLoop 可以与多个 Channel 绑定，但一个 Channel 只能绑定一个 EventLoop

Channel(n)--------(1)EventLoop(1) --------(1)Thread

#### EventLoopGroup

 EventLoopGroup 是一个 EventLoop 池，包含很多的 EventLoop。NioEventLoopGroup相当于一个事件循环组,这个组中含有**多个事件循环**，每一个事件循环是 NioEventLoop。可以有多个线程,即可以含有多个NioEventLoop





Netty抽象出**两组线程池**parentGroup专门负责连接accept，childGroup专门负责网络的读写、业务处理
BossGroup和 WorkoerGroup类型都是**NioEventLoopGroup**

NloEventLoopGroup.每个parentGroup循环执行的步骤有3步

轮询accept事件
处理accept事件，与cllent建立连接,生成NloScocketChannel,并将其注册到某个worker NIOEventLoop 上的 selector
处理任务队列的任务,即runAllTasks

每个childGroup不断执行的步骤轮询read, write事件
处理i/o事件,即read , write事件,在对应NioScocketChannel处理处理任务队列的任务,即runAllTasks
每个childGroup处理业务时,会使用pipeline(管道),pipeline中包含了channel,即通过pipeline可以获取到对应通道,管道维护很多handler



#### 联系

NioEventLoop内部采用串行化设计，从消息的读取->解码->处理->编码->发送，始终由I0线程NioEventLoop负责

NioEventLoopGroup下包含多个NioEventloop
每个NioEventLoop中包含有一个Selector，一个taskQueue每个NioEventLoop的Selector上可以注册监听多个NioChannel每个NioChannel只会绑定在唯一的 NioEventLoop 上
每个 NioChannel都绑定有一个自己的 ChannelPipeline

### ServerBootstrap 与 Bootstrap

 Bootstarp 和 ServerBootstrap 被称为引导类，指对应用程序进行配置，并使他运行起来的过程。Netty处理引导的方式是使你的应用程序和网络层相隔离。

 Bootstrap 是客户端的引导类，Bootstrap 在调用 bind()（连接UDP）和 connect()（连接TCP）方法时，会新创建一个 Channel，仅创建一个单独的、没有父 Channel 的 Channel 来实现所有的网络交换。

 ServerBootstrap 是服务端的引导类，ServerBootstarp 在调用 bind() 方法时会创建**一个 ServerChannel** 来接受来自客户端的**连接**，并且该 ServerChannel 管理了多个**子 Channel 用于同客户端**之间的通信。

### ChannelHandler 与 ChannelPipeline

 ChannelHandler 是对 Channel 中数据的处理器，这些处理器可以是系统本身定义好的编解码器，也可以是用户自定义的。这些处理器会被统一添加到一个 ChannelPipeline 的对象中，然后按照添加的顺序对 Channel 中的数据进行依次处理。

### ChannelFuture

 Netty 中所有的 I/O 操作都是异步的，即操作不会立即得到返回结果，所以 Netty 中定义了一个 ChannelFuture 对象作为这个异步操作的“代言人”，表示异步操作本身。如果想获取到该异步操作的返回值，可以通过该异步操作对象的addListener() 方法为该异步操作添加监 NIO 网络编程框架 Netty 听器，为其注册回调：当结果出来后马上调用执行。

 Netty 的异步编程模型都是建立在 Future 与回调概念之上的。
