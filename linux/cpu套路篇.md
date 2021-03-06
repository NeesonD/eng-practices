### 如何快速分析出 CPU 瓶颈

观察图中的指标

![](pic/cpu.png)
 

**性能工具**

如果有异常指标，则通过相应的性能工具去分析，最终目的是为了定位问题所在
![](pic/cpu-tools.png)

**案例**

* 平均负载案例
* 上下文切换案例
* 进程 CPU 使用率升高案例
* 系统 CPU 使用率升高案例
* 不可中断进程和僵尸进程案例
* 软中断案例

![](pic/cpu-tools2.png)

* 想弄清楚性能指标的关联性，就要通晓每种性能指标的工作原理
* 缩小排查范围，我通常会先运行几个支持指标较多的工具

![](pic/cpu-tools3.png)

* 从 top 的输出可以得到各种 CPU 使用率以及僵尸进程和平均负载等信息
* 从 vmstat 的输出可以得到上下文切换次数、中断次数、运行状态和不可中断状态的进程数
* 从 pidstat 的输出可以得到进程的用户 CPU 使用率、系统 CPU 使用率、以及自愿上下文切换和非自愿上下文切换情况


### 如何优化

性能优化方法论

* 确定性能的量化指标
    * 应用程序的维度，我们可以用**吞吐量**和**请求延迟**来评估应用程序的性能
    * 系统资源的维度，我们可以用 **CPU 使用率**来评估系统的 CPU 使用情况
    * 原因
        * 好的应用程序是性能优化的最终目的和结果，系统优化总是为应用程序服务的。所以，必须要使用应用程序的指标，来评估性能优化的整体效果
        * 系统资源的使用情况是影响应用程序性能的根源。所以，需要用系统资源的指标，来观察和分析瓶颈的来源
* 测试优化前的性能指标
* 测试优化后的性能指标
    * 第一，要避免性能测试工具干扰应用程序的性能
    * 第二，避免外部环境的变化影响性能指标的评估