# Reactor

IO 多路复用的核心：

* 当多条连接共用一个阻塞对象后，进程只需要在一个阻塞对象上等待，而无须再轮询所有连接，常见的实现方式有 select、epoll、kqueue 等
* 当某条连接有新的数据可以处理时，操作系统会通知进程，进程从阻塞状态返回，开始进行业务处理

本质上是从轮询变成了事件通知

Reactor 三种模式：

## 单 Reactor 单进程 / 线程

![单 Reactor 单进程 / 线程](pic/single-reactor-single-thread.png)

* Reactor 对象通过 select 监控连接事件，收到事件后通过 dispatch 进行分发
* 如果是连接建立的事件，则由 Acceptor 处理，Acceptor 通过 accept 接受连接，并创建一个 Handler 来处理连接后续的各种事件
* 如果不是连接建立事件，则 Reactor 会调用连接对应的 Handler（第 2 步中创建的 Handler）来进行响应
* Handler 会完成 read-> 业务处理 ->send 的完整业务流程

### 单 Reactor 单进程 / 线程缺点

* 只有一个进程，无法发挥多核 CPU 的性能；只能采取部署多个系统来利用多核 CPU，但这样会带来运维复杂度，本来只要维护一个系统，用这种方式需要在一台机器上维护多套系统
* Handler 在处理某个连接上的业务时，整个进程无法处理其他连接的事件，很容易导致性能瓶颈

> 只适用于业务处理非常快速的场景，比如说 redis (单 Reactor 单进程)

## 单 Reactor 多线程

![单 Reactor 多线程](pic/single-reactor-muli-thread.png)

* 主线程中，Reactor 对象通过 select 监控连接事件，收到事件后通过 dispatch 进行分发
* 如果是连接建立的事件，则由 Acceptor 处理，Acceptor 通过 accept 接受连接，并创建一个 Handler 来处理连接后续的各种事件
* 如果不是连接建立事件，则 Reactor 会调用连接对应的 Handler（第 2 步中创建的 Handler）来进行响应
* Handler 只负责响应事件，不进行业务处理；Handler 通过 read 读取到数据后，会发给 Processor 进行业务处理
* Processor 会在独立的子线程中完成真正的业务处理，然后将响应结果发给主进程的 Handler 处理；Handler 收到响应后通过 send 将响应结果返回给 client

### 单 Reactor 多线程缺点

* 多线程数据共享和访问比较复杂。例如，子线程完成业务处理后，要把结果传递给主线程的 Reactor 进行发送，这里涉及共享数据的互斥和保护机制。以 Java 的 NIO 为例，Selector 是线程安全的，但是通过 Selector.selectKeys() 返回的键的集合是非线程安全的，对 selected keys 的处理必须单线程处理或者采取同步措施进行保护
* Reactor 承担所有事件的监听和响应，只在主线程中运行，瞬间高并发时会成为性能瓶颈

## 多 Reactor 多进程 / 线程

![多 Reactor 多进程 / 线程](pic/muli-reactor-muli-thread.png)

* 父进程中 mainReactor 对象通过 select 监控连接建立事件，收到事件后通过 Acceptor 接收，将新的连接分配给某个子进程
* 子进程的 subReactor 将 mainReactor 分配的连接加入连接队列进行监听，并创建一个 Handler 用于处理连接的各种事件
* 当有新的事件发生时，subReactor 会调用连接对应的 Handler（即第 2 步中创建的 Handler）来进行响应
* Handler 完成 read→业务处理→send 的完整业务流程

### 优点

* 父进程和子进程的职责非常明确，父进程只负责接收新连接，子进程负责完成后续的业务处理
* 父进程和子进程的交互很简单，父进程只需要把新连接传给子进程，子进程无须返回数据
* 子进程之间是互相独立的，无须同步共享之类的处理（这里仅限于网络模型相关的 select、read、send 等无须同步共享，“业务处理”还是有可能需要同步共享的）

### 应用

* Nginx 采用的是多 Reactor 多进程
* Netty 多 Reactor 多线程

-----

IO操作分两个阶段
1、等待数据准备好(读到内核缓存)
2、将数据从内核读到用户空间(进程空间)
一般来说1花费的时间远远大于2。
1上阻塞2上也阻塞的是同步阻塞IO
1上非阻塞2阻塞的是同步非阻塞IO，这讲说的Reactor就是这种模型
1上非阻塞2上非阻塞是异步非阻塞IO，这讲说的Proactor模型就是这种模型