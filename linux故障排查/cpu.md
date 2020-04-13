### 基础篇

#### 关健指标一：平均负载

平均负载指的是，**单位时间**内的平均**活跃进程数**（可运行状态和不可中断状态），和 CPU 没有直接关系

可运行状态进程：R，正在使用或者正在等待 CPU 的进程
不可中断状态进程：正处于内核态关健流程中的进程，且不可打断。最常见的就是**等待硬件 IO 响应**

**该指标为多少时是合理的**

* 确认系统 CPU 个数
```shell script
grep 'model name' /proc/cpuinfo | wc -l
```
* 查看平均负载
```shell script
uptime
                                                    1min  5min  15min
18:24:16 up 41 days,  6:38, 1 users,  load average: 1.70, 1.58, 1.60
```
通过分析三个负载值，可以看出系统负载是平稳的，还是降低或者升高的

* 当平均负载高于 CPU 数量 70% 的时候，就需要分析负载高的问题了；或者当系统 CPU 出现突刺的时候
    * CPU 密集型进程，使用大量 CPU 会导致平均负载升高；
    * **IO 密集型进程，等待 IO 也会导致平均负载升高**，但是 CPU 使用率不一定高
    * 大量等待 CPU 的进程调度也会导致平均负载高，此时 CPU 的使用率也会比较高
 
#### 实操演示
* stress: 压测工具
* sysstat：性能监控工具

```shell script
yum install stress sysstat
```

##### 模拟 CPU 密集型应用

```shell script
## 压力测试
stress --cpu 1 --timeout 600

## 负载变化
watch -d uptime

## 查看 cpu 使用率的变化
mpstat -P ALL 5

## 查看是哪个进程导致了 CPU 升高
pidstat -u 5 1
```

##### 模拟 IO 密集型应用