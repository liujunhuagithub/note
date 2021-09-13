

# 协程Coroutine

可看作用户态轻量级线程。将**异步+回调**的**复杂写法**，抽象为顺序**串行**的表达，**底层库处理异步性**。适用于IO密集型、嵌套回调、并发编程。本质上还是回调

**一个线程有多个协程，但同一时刻只有一个协程运行**

协程要维护**局部的上下文**，重新进入上次挂起的位置，上下文恢复不变

协程**挂起**suspend**不会阻塞线程**，几乎无代价，完全由程序控制，无需OS干预，

挂起时，线程回收到池中；结束挂起，在池中空闲的线程恢复执行。使用**线程池模拟协程实现**

## 开启协程

### lanuch

创建协程/子协程：`val v:Job=scope.launch(context，start，block) { }`  

启动**新**协程，返回持有协程引用的**Job**任务。不指定则启动本作用域协程

- 不指定scope和context，**默认**作为父的**子协程**
- 返回的**Job没有包含返回值**
- 可在`suspend`函数内启动

**多个子协程并行运行，单个子协程内代码同步**

##### Job状态

| 状态                       | isActive | isCompleted |
| -------------------------- | -------- | ----------- |
| New新建(可选的初始状态)    | ❌        | ❌           |
| Active活动中(默认初始状态) | ✔        | ❌           |
| Completed已结束(最终状态)  | ❌        | ✔           |

job.`join`( )：**协程域内显式**等待某job运行完毕，**期间挂起当前协程**，等待子协程运行完毕。**非阻塞线程**



### async(suspend函数异步并行)

默认`suspend`函数耗时操作**顺序串行**某些业务可并行，可改为异步操作

`async (context，start，block) {}`：**创建子协程**，与其他协程**并行**，

- 返回`Deferred`(Job子类，轻量级非阻塞可取消future) ，**包含结果**
- 可在`suspend`函数内启动
- 懒启动`start=CoroutineStart.LAZY`：协程不自动启动，只有 ①显式`start()` ②`await( )`才启动
  - 仅await( )可能导致顺序**串行执行**

`await()`：延期返回`async`的`suspend`函数运行完毕的结果。常常try-catch在此处处理异常

#### 异步风格

`GlobalScope.async {   }`：作为**普通函数**，非suspoend。任何地方使用。用于异步调用

`runBlocking` 中阻塞主线程调用`await`返回结果

#### Deferred状态

| 状态           | isActive | isCompleted | isCompletedExceptionally | isCancelled |
| -------------- | -------- | ----------- | ------------------------ | ----------- |
| New可选初始    | ❌        | ❌           | ❌                        | ❌           |
| Active默认初始 | ✔        | ❌           | ❌                        | ❌           |
| Resolved       | ❌        | ✔           | ❌                        | ❌           |
| Failed         | ❌        | ✔           | ✔                        | ❌           |
| Cancelled      | ❌        | ✔           | ✔                        | ✔           |

`isCompletedExceptionally`：执行异常、cancel

`isCancelled`：任务取消



### suspend挂起函数

**`suspend`**修饰：执行**完毕自动恢复**resume，函数体**启动新协程**。常用于执行耗时操作

kotlinx.coroutines内置，`delay`( ):**挂起协程**指定时间，**协程内使用**

只能在**协程体**、**suspend函数**内调用。不能使用Thread启动



## 协程配置

- `context`：协程上下文 。显式的为一个新协程或其它上下文元素指定一个调度器
- `start` ：启动选项  默认 CoroutineStart.Default
- `block`：业务代码块，suspend修饰



### 协程上下文CoroutineContext

协程调度器：确定相应的协程使用哪个或哪些线程来执行。**默认无指定父协程上下文**

Dispatchers是CoroutineContext的实现

| 调度器CoroutineDispatcher      | 作用域                                            | 特点                                                         |
| ------------------------------ | ------------------------------------------------- | ------------------------------------------------------------ |
| Dispatcher.Unconfined          | 非受限调度器，继承父作用域                        | 仅仅只是运行到第一个挂起点。挂起后，它恢复回哪个线程执行，这完全由被调用的挂起函数来决定适用于更新UI线程、不占用CPU的任务 |
| Dispatcher.Default             | 使用ForkJoin共享线程池，`GlobalScope`默认的调度器 | 计算型任务(解析数据)                                         |
| Dispatcher.IO                  | 非主线程，使用ForkJoin共享线程池                  | 数据库、网络等IO                                             |
| Dispatcher.Main                | UI主线程                                          | 更新UI、轻量级suspend                                        |
| newSingleThreadContext("name") | 新建指定线程作用域                                |                                                              |

各个Job子协程：从rou

coroutineContext[ Job ]获取

### 协程作用域CoroutineScope

协程只能在**指定的CoroutineScope **中启动。

结构化并发：多个分开的并发路径最终再次连接起来，使得符合因果关系。<u>不同CoroutineScope中的协程具有不同的声明周期</u>，能够控制内部协程的生命周期，可以取消内部所有协程

- 创建作用域：`CoroutineScope(指定scope)`
- 作用域特点：
  - 能够控制内部协程的生命周期
  - 可以取消内部所有协程
  - 所有子协程完成后作用域才结束

- cancel()：用于取消当前作用域，同时**取消作用域内所有协程**。

#### 非协程环境启动

- `GlobalScope.lanuch`：作为顶层协程(进程process级)，与启动的作用域独立无关，**没有父协程**。
  - APP束时也会跟着一起结束。不会使进程保活，**如同守护线程**。不能取消
  - 单例对象，含有一个空的上下文对象。使用Dispatcher.Default调度器
  - 慎用
- `runBlocking{ }`：**代码块阻塞**当前**线程**，直到**此协程体、子协程执行完毕**才继续执行runBlocking下面的，占用线程资源
  - 主协程，只能启动其他并发的新协程，不能别其他协程启动。默认使用Dispatcher.Unconfined调度器
  - 常规函数。用于桥接阻塞和非阻塞代码，很少使用
  - 由于**本身阻塞**，故而代码**块内**的**挂起**操作**对域外**表现为**线程阻塞**
- `某作用域.launch`：新建自定义域

#### 协程内启动

- `launch/async{ }`：**默认**不指定：继承父协程作用域，**作为父协程的并行子协程**
  - 本身是普通函数，**父协程一直运行，不会挂起**
- `coroutineScope{ }`自定义作用域构建器：创建新作用域，**并不启动协程**
  - 本身是`suspend`函数，**父协程一直挂起**
  - 用于将各种相互依赖**子任务组合为大的父任务**，用于自定义结构化并发
  - **不阻塞线程**。释放线程资源
- `withContext( scope) { }`：切换指定作用域运行，将长耗时操作从UI线程切走，完事再切回来
  - 本身是`suspend`函数，**父协程一直挂起**
  - 并不创建新协程

```kotlin
fun main() = runBlocking {
    doWorld()
}

suspend fun doWorld() = coroutineScope {  // this: CoroutineScope
    launch {
        delay(1000L)
        println("World!")
    }
    println("Hello")
}
```

![image-20210831163854369](F:\学习总结\kotlin\image-20210831163854369.png)

#### 父子协程

**默认不指定**CoroutineContext启动的协程则为子协程，指定其他作用域启动的新协程不是子协程

==**父协程取消，子协程会递归地取消；**==

==**子协程抛出异常`非CancellationException`不处理，整个父协程作用域都会被自动取消**==

==**父协程/作用域自动等待  所有子协程、代码块运行完毕，才结束**==。其他作用域协程需显式join

## 协程取消

### cancel()

`job.cancel( )`：取消协程。返回true，再次取消返回false

协程处于**挂起suspend**状态才能**真正停止执行**，占用线程CPU资源仅仅状态改变取消

`cancelAndJoin`：**cancel并join**，触发cancel并**等待计算任务**执行、`finally`完毕

结束协程：return@launch

**父协程取消，子协程会递归地取消；子协程抛出异常，整个父协程作用域都会被自动取消**

| cancel状态       | isActive | isCompleted |
| ---------------- | -------- | ----------- |
| cancel前         | ✔        | ❌           |
| cancel正在取消中 | ❌        | ❌           |
| cancel完成       | ❌        | ✔           |

![在这里插入图片描述](F:\学习总结\kotlin\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pvdTg5NDQ=,size_16,color_FFFFFF,t_70)

### 取消失效

协程处于**非suspend**状态(**占用线程**资源)时`cancel`，仅仅**状态变化**，**不会真正停止执行**。

- 构造**可取消协程**
  - 协程业务代码**显式检查isActive**标志位，isActive=false则`return@launch`
  - **循环**调用**yield**( )并捕获异常：**cancel后**(isCompleted=true)时自动**抛出**`CancellationException`
- **父协程取消，子协程会递归地取消；**
- `withTimeout( L)`：超时自动取消，抛出`TimeoutCancellationException`异常  。需要返回结果需**建立外部变量**在协程内部赋值，并finally关闭资源。`withTimeoutOrNull( L)`不报错返回null
- `withContext(NonCancellable)`{ }内调用suspend函数不受cancel影响。`finally`不能直接调用`suspend`函数，此时协程已取消不会被调度执行。不常用

```kotlin
while (isActive) { // cancellable computation loop
        if (System.currentTimeMillis() >= nextPrintTime) {
            println("job: I'm sleeping ${i++} ...")
            nextPrintTime += 500L
        }
    }
if(!isActive) return@launch

    while () { // cancellable computation loop
        yield()
        if (System.currentTimeMillis() >= nextPrintTime) {
            println("job: I'm sleeping ${i++} ...")
            nextPrintTime += 500L
        }
    }
```

### 异常处理

**协程被cancel 时在挂起点`suspend`抛出`CancellationException`**

#### 异常捕获

- `launch`：kotlin默认**忽略**launch抛出的`CancellationException`，不输出此异常信息。用户无需关心
- `async`：用户在**`await( )处`**自定义`try-catch`，kotlin**不会忽略**，输出此异常信息

1. **子协程cancel产生的CancellationException, 不会影响父协程。**
2. **子协程抛出非CancellationException异常不处理，父协程将会因异常而取消, 牵连父协程域内的所有子协程**

## 多线程调度器并发

协程可用多线程调度器（比如默认的 `Dispatchers.Default`）并发执行。这样就会有多线程中的并发问题

@Volatile 声明共享同步变量   @Synchornized  同步函数



## 同道Channel

**协程间传输数据流**，**延时**等待数据`send( ) / receive( )`。suspend协程而非阻塞线程

```kotlin
val c=Channel<Int>()
c.send(12)
c.receive()
for(i in c){ }//直到close
```

### 通道关闭

close( )：相当于发送特别的令牌，重复调用无效。

发送端isClosedForSend=true，接收端收到元素后isClosedReceive=true

接受全部已发送的数据后接收端才关闭

关闭后不能send / receive数据，否则报错

### 生产者-消费者

生产者抽象为以Channel为核心的**函数**，生产者**结果从函数返回**

`produce( context,capacity缓存默认0,block)`

**启动新协程**，返回ProducerJob，是`ReceiveChannel`

```kotlin
fun CoroutineScope.numbersFrom(start: Int) = produce<Int> {
    var x = start
    while (true) send(x++) // infinite stream of integers from start
}
```

