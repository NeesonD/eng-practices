### 进程不断重启

```
 docker run --name nginx -p 10000:80 -itd feisky/nginx:sp
 docker run --name phpfpm -itd --network container:nginx feisky/php-fpm:sp

 # 并发5个请求测试Nginx性能，持续 10min
 ab -c 5 -t 600 http://192.168.0.10:10000/


 # 通过top和pidstat找不到占用率高的进程，使用 pstree 找父进程
 pstree | grep stress

 # 记录性能事件，等待大约15秒后按 Ctrl+C 退出
 perf record -g

 # 查看报告
 perf report

 perf record -ag -- sleep 2;perf report

```

* 碰到**常规问题无法解释的 CPU 使用率情况**时，首先要想到有可能是**短时应用导致的问题**，比如有可能是下面这两种情况。
* 第一，应用里直接调用了其他二进制程序，这些程序通常运行时间比较短，通过 top 等工具也不容易发现。
* 第二，应用本身在不停地崩溃重启，而启动过程的资源初始化，很可能会占用相当多的 CPU。对于这类进程，我们可以用 pstree 或者 execsnoop 找到它们的父进程，再从父进程所在的应用入手，排查问题的根源


### 不可中断进程

```
top - 21:22:23 up 61 days, 27 min,  4 users,  load average: 3.16, 5.86, 5.18
Tasks: 714 total,   1 running, 713 sleeping,   0 stopped,   0 zombie
%Cpu(s):  9.3 us,  8.3 sy,  0.0 ni, 82.1 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  8010688 total,   529244 free,  2197940 used,  5283504 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  3536908 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                              
  742 root      20   0 1472072 138064  18224 S   3.0  1.7   1508:15 kubelet                                                                                   


  * S 列（也就是 Status 列）表示进程的状态    
    * R 是 Running 或 Runnable 的缩写，表示进程在 CPU 的就绪队列中，正在运行或者正在等待运行
    * D 是 Disk Sleep 的缩写，也就是不可中断状态睡眠（Uninterruptible Sleep），一般表示进程正在跟硬件交互，并且交互过程不允许被其他进程或中断打断
    * Z 是 Zombie 的缩写，如果你玩过“植物大战僵尸”这款游戏，应该知道它的意思。它表示僵尸进程，也就是进程实际上已经结束了，但是父进程还没有回收它的资源（比如进程的描述符、PID 等）
    * S 是 Interruptible Sleep 的缩写，也就是可中断状态睡眠，表示进程因为等待某个事件而被系统挂起。当进程等待的事件发生时，它会被唤醒并进入 R 状态
    * I 是 Idle 的缩写，也就是空闲状态，用在不可中断睡眠的内核线程上。前面说了，硬件交互导致的不可中断进程用 D 表示，但对某些内核线程来说，它们有可能实际上并没有任何负载，用 Idle 正是为了区分这种情况。要注意，D 状态的进程会导致平均负载升高， I 状态的进程却不会
```

* 如果系统或硬件发生了故障，进程可能会在不可中断状态保持很久，甚至导致系统中出现大量不可中断进程。这时，你就得注意下，系统是不是出现了 I/O 等性能问题
* 父进程还没来得及处理子进程状态，子进程就已经提前退出，那这时的子进程就会变成僵尸进程

```
docker run --privileged --name=app -itd feisky/app:iowait


```

