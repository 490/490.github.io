---
title: Java虚拟机性能监控及调优
date: 2019-03-11 10:51:16
tags: [Java,JVM]
---
# 常用虚拟机性能监控工具
## JDK 命令行工具
![image](http://490.github.io/images/20190311_105149.png)
其中的重中之重是 jstat 命令！而它最常用的参数就是 -gcutil，使用格式如下：
```source-shell
jstat -gcutil [pid] [intervel] [count]
```
<!--more-->
输出如下：

*   `S0`：堆上 Survivor space 0 区已使用空间的百分比
*   `S1`：堆上 Survivor space 1 区已使用空间的百分比
*   `E`：堆上 Eden 区已使用空间的百分比
*   `O`：堆上 Old space 区已使用空间的百分比
*   `P`：堆上 Perm space 区已使用空间的百分比
*   `YGC`：从程序启动到采样时发生的 Minor GC 次数
*   `YGCT`：从程序启动到采样时 Minor GC 所用的时间
*   `FGC`：从程序启动到采样时发生的 Full GC 次数
*   `FGCT`：从程序启动到采样时 Full GC 所用的时间
*   `GCT`：从程序启动到采样时 GC 的总时间
## ps 命令 (Linux)
对于 `jps` 命令，其实没必要使用，一般使用 Linux 里的 `ps` 就够了，`ps` 为我们提供了当前进程状态的一次性的查看，它所提供的查看结果并不动态连续的，如果想对进程时间监控，应该用 `top` 工具。

**Linux 上进程的 5 种状态**

*   运行 [R, Runnable]：正在运行或者在运行队列中等待；
*   中断 [S, Sleep]：休眠中, 受阻, 在等待某个条件的形成或接受到信号；
*   不可中断 [D]：收到信号不唤醒和不可运行, 进程必须等待直到有中断发生；
*   僵死 [Z, zombie]：进程已终止, 但进程描述符存在, 直到父进程调用 wait4() 系统调用后释放；
*   停止 [T, Traced or stop]：进程收到 SIGSTOP, SIGSTP, SIGTIN, SIGTOU 信号后停止运行运行。

```source-shell
ps -A # 列出所有进程信息（非详细信息）
ps aux  # 列出所有进程的信息
ps aux | grep zsh

ps -ef # 显示所有进程信息，连同命令行
ps -ef | grep zsh 

ps -u root # 显示指定用户信息
ps -l  # 列出这次登录bash相关信息

ps axjf  # 同时列出进程树状信息
```
# JVM 常见参数设置
## 内存设置

### 参数

- `-Xms`：初始堆大小，JVM 启动的时候，给定堆空间大小。
- `-Xmx`：最大堆大小，如果初始堆空间不足的时候，最大可以扩展到多少。
- `-Xmn`：设置年轻代大小。`整个堆大小 = 年轻代大小 + 年老代大小 + 持久代大小`。持久代一般固定大小为 64M，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun 官方推荐配置为整个堆的 3/8。
- `-Xss`： 设置每个线程的 Java 栈大小。JDK 5 后每个线程 Java 栈大小为 1M。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在 3000~5000 左右。
- `-XX:NewRatio=n`：设置年轻代和年老代的比值。如为 3，表示年轻代与年老代比值为 1:3。
- `-XX:MaxTenuringThreshold`：设置垃圾最大年龄。如果设置为 0 的话，则年轻代对象不经过 Survivor 区，直接进入年老代。对于年老代比较多的应用（即 Minor GC 过后有大量对象存活的应用），可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在 Survivor 区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概率。

### 设置经验

- 开发过程的测试应用，要求物理内存大于 4G

```
	-Xmx3550m
	-Xms3550m 
	-Xmn2g
	-Xss128k
```

- 高并发本地测试使用，大对象相对较多（如 IO 流）
```
	-Xmx3550m
	-Xms3550m
	-Xss128k
	-XX:NewRatio=4
	-XX:SurvivorRatio=4
	-XX:MaxPermSize=160m
	-XX:MaxTenuringThreshold=0
```
- 环境： 16G 物理内存，高并发服务，重量级对象中等（线程池，连接池等），常用对象比例为 40%（即运行过程中产生的对象 40% 是生命周期较长的）

```
	-Xmx10G
	-Xms10G
	-Xss1M
	-XX:NewRatio=3
	-XX:SurvivorRatio=4 
	-XX:MaxPermSize=2048m
	-XX:MaxTenuringThreshold=5
```

## 收集器设置

### 参数

- 收集器设置
	- `-XX:+UseSerialGC`：设置串行收集器，年轻带收集器。
	- `-XX:+UseParallelGC`：设置并行收集器。
	- `-XX:+UseParNewGC`：设置年轻代为并行收集。可与 CMS 收集同时使用。JDK 5.0 以上，JVM 会根据系统配置自行设置，所以无需再设置此值。
	- `-XX:+UseParallelOldGC`：设置并行年老代收集器，JDK6.0 支持对年老代并行收集。
	- `-XX:+UseConcMarkSweepGC`：设置年老代并发收集器，测试中配置这个以后，`-XX:NewRatio` 的配置失效，原因不明。所以，此时年轻代大小最好用 `-Xmn` 设置。
	- `-XX:+UseG1GC`：设置 G1 收集器。
- 并行收集器参数设置
	- `-XX:ParallelGCThreads=n`：设置并行收集器收集时最大线程数使用的 CPU 数。并行收集线程数。
	- `-XX:MaxGCPauseMillis=n`：设置并行收集最大暂停时间，单位毫秒。
	- `-XX:GCTimeRatio=n`：设置垃圾回收时间占程序运行时间的百分比。
	- `-XX:+UseAdaptiveSizePolicy`：设置此选项后，并行收集器会自动选择年轻代区大小和相应的 Survivor 区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。
	- `-XX:CMSFullGCsBeforeCompaction=n`：由于 CMS 不对内存空间进行压缩、整理，所以运行一段时间以后会产生"碎片"，使得运行效率降低。此值设置运行多少次 GC 以后对内存空间进行压缩、整理。
	- `-XX:+UseCMSCompactAtFullCollection`：打开对年老代的压缩。可能会影响性能，但是可以消除碎片。
# 虚拟机调优案例分析
## 高性能硬件上的程序部署策略
![image](http://490.github.io/images/20190311_105436.png)
**补充：64 位虚拟机**
在 Java EE 方面，企业级应用经常需要使用超过 4GB 的内存，此时，32 位虚拟机将无法满足需求，可是 64 位虚拟机虽然可以设置更大的内存，却存在以下缺点：
*   **内存问题：** 由于指针膨胀和各种数据类型对齐补白的原因，运行于 64 位系统上的 Java 应用程序需要消耗更多的内存，通常要比 32 位系统额外增加 10% ~ 30% 的内存消耗。
*   **性能问题：** 64 位虚拟机的运行速度在各个测试项中几乎全面落后于 32 位虚拟机，两者大概有 15% 左右的性能差距。

## 服务系统经常出现卡顿（Full GC 时间太长）

首先 `jstat -gcutil` 观察 GC 的耗时，`jstat -gccapacity` 检查内存用量（也可以加上 `-verbose:gc` 参数获取 GC 的详细日志），发现卡顿是由于 Full GC 时间太长导致的，然后 `jinfo -v pid`，查看虚拟机参数设置，发现 `-XX:NewRatio=9`，这就是原因：

- 新生代太小，对象提前进入老年代，触发 Full GC
- 老年代较大，一次 Full GC 时间较长

可以调小 NewRatio 的值，尽肯能让比较少的对象进入老年代。

## 除了 Java 堆和永久代之外，会占用较多内存的区域

| 区域          | 大小调整 / 说明                                             | 内存不足时抛出的异常                                         |
| ------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| 直接内存      | `-XX:MaxDirectMemorySize`                                   | OutOfMemoryError: Direct buffer memory                       |
| 线程堆栈      | `-Xss`                                                      | StackOverflowError 或 OutOfMemoryError: unable to create new native thread |
| Socket 缓存区 | 每个 Socket 连接都有 Receive(37KB) 和 Send(25KB) 两个缓存区 | IOException: Too many open files                             |
| JNI 代码      | 如果代码中使用 JNI 调用本地库，那本地库使用的内存也不在堆中 |                                                              |
| 虚拟机和 GC   | 虚拟机、GC 代码执行要消耗一定内存                           |                                                              |

## 从 GC 调优角度解决新生代存活大量对象问题（Minor GC 时间太长）

- 将 Survivor 空间去除，让新生代中存活的对象在第一次 Minor GC 后立刻进入老年代，等到 Full GC 时再清理。
- 参数调整方法：
	- `-XX:SurvivorRatio=65536`
	- `-XX:MaxTenuringThreshold=0`
	- `-XX:AlwaysTenure`

