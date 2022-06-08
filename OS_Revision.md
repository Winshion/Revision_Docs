#### 第一章



操作系统发展流程：<u>手工操作、批处理、多道程序操作系统、分时操作系统、实时操作系统</u>。

#### 第二章

冯诺依曼体系架构的完整描述：<u>计算机由运算单元、控制单元、输入、输出、存储器构成，其中以存储器为核心</u>。

系统启动的表述：从外存中导入操作系统内核至内存，然后操作系统控制权被转移至基础环境。整个过程成为<u>bootstrap（boot）</u>。

Windows XP以前的bootstrap程序：NTLDR；Vista之后的成为BOOTMGR；Linux/Unix系统的bootloader称为GRUB。

CPU访问外设通过<u>设备控制器</u>。

CPU访问外设的方法有：<u>I/O端口号【独立编址】、内存映射I/O【统一编址】</u>。

中断分为<u>硬件中断和软件中断</u>，其中软件中断又称为<u>异常</u>。

<font color='red'>陷入操作系统内核态的两种方式：<u>系统调用、特权指令</u>。</font>

硬件保护的基本方式：<u>二态模式、特权指令分级、内存保护、CPU保护【通过定时器】</u>。

系统调用：操作系统和用户程序之间的接口，操作系统为用户提供服务的最小功能单位，用户使用<u>API</u>编写程序。

#### 第三章

操作系统结构分为<u>简单结构（DOS、UNIX）、层次结构（THE、IBM OS/2）、微内核（Windows、Mach）、宏内核（Linux）</u>。

操作系统设计的用户目标：<u>Convenient to use, easy  to learn, reliable, safe and fast</u>；系统目标：<u>Easy to design, implement and maintain, reliable, flexible, error-free and efficient.</u>

操作系统设计原则：策略（做什么）与机制（怎么做）分离。

#### 第四章

进程包括<u>代码、数据、堆栈和堆</u>，是正在运行的程序或任务的抽象。

【超级重要】进程的状态迁徙图（默写下来），其中由就绪态转换到运行态的为短期调度，由新建态转换到就绪态为长期调度。

非抢占性：running->terminated，running->waiting；抢占性：running->ready，new->ready。

进程控制块：保存<u>进程号、进程状态、程序指针、I/O控制信息、内存管理信息、CPU寄存器(最重要)、CPU调度信息、记账信息（Accounting info, 翻译得不好）</u>

上下文切换：指CPU切换到另一个进程时，保存当前进程状态，并恢复另一进程状态的过程。

进程调度队列分为<u>作业队列、就绪队列</u>。

有限缓冲区的生产者-消费者问题：

```c
#define BUFSIZE 10
int in = 0, out = 0;
int buffer[BUFSIZE];

// Producer
while (1) {
    /* 生产一个元素 */
    while ((in + 1) % BUFSIZE == out);
    buffer[in] = itemProduced;
    in = (in + 1) % BUFSIZE;
}


// Consumer
while (1) {
    while ((out + 1) % BUFSIZE == in);
    itemGet = buffer[out];
    out = (out + 1) % BUFSIZE;
    /* 消费一个元素 */
}
```

进程间通信通过<u>管道</u>实现，主要分为命名管道和匿名管道。

#### 第六章

进程按照主要运行时间差异，分为CPU绑定进程和I/O绑定进程。

调度算法：FCFS、SJF（公平）、Round Robin（取决于时间片，短了上下文切换过于频繁、长了就变成FCFS）和Priority Queue

CPU调度KPI：CPU利用率、吞吐量、周转时间、等待时间、响应时间

第七版课后习题5.4：

* FCFS：12345，等待时间和为48，平均等待时间9.6

* SJF：24351，等待时间和为16，平均等待时间3.2

* 非抢占优先级：25134，等待时间和41，平均等待时间8.2

* RR（时间片=1）：1234513515151511111，等待时间和26，平均5.2

进程分队列调度（队列间不转换）：多级队列；（队列间可以相互转移）：多级反馈队列

优先级调度会产生的问题是<u>饥饿、优先级反转</u>，解决方式为<u>老化（Aging，翻译得不好）</u>

#### 第七章

竞争条件：并发进程或线程对共享变量的访问导致数据不一致的情况

解决竞争条件必须满足的三个条件：互斥性、前进性（有空让进）、有限等待

解决竞争条件：

* 硬件：使用test_and_set原语和swap原语【不满足<font color='red'>有限等待性</font>】

* 软件：使用进程同步算法

* 上层（高级语言）：管程

【重要】Peterson算法

```cpp
int turn = 0;
bool flag[2] = { false, false };
int pid = 0;    // or 1

void schedule(int pid) {
    flag[i] = true;
    turn = (pid + 1) % 2;
    while (turn == (pid + 1) % 2 && flag[(i + 1) % 2]);    // 【重要】
    /* Critical Section */
    flag[i] = false;
}


while (1) { schedule(pid); }
```

【重要】信号灯，value为正代表可用资源数，为负代表在就绪队列中排队的进程

```cpp
class PCB {
    /* PCB Definition */
}
class Semaphore {
    int value;
    PCB* queue;
};
```

调度信号灯的两个操作：P（wait）和V（signal）：

```cpp
void P(Semaphore* S) {
    S->value--;
    if (S->value < 0)    // 这里很重要！
        /* 挂起进程 */
        hang_up(currproc);
}

void P(Semaphore* S) {
    S->value++;
    if (S->value <= 0)    // 这里很重要！
        /* 唤醒进程 */
        wake_up(currproc);
}
```

哲学家进餐问题：无法避免饥饿现象。

【重要】读者-写者问题代码

```cpp
Semaphore write, mutex;
int readcount;
// 作者
while (1) {
    P(&write);
    writing();
    V(&write);
}

// 读者：mutex信号灯为了保护变量readcount，因此mutex的value值应该初始化
while (1) {
    P(&mutex);
    readcount++;
    if (readcount == 1)
        P(&write);
    V(&mutex);

    reading();
    
    P(&mutex);
    readcount--;
    if (readcount == 0)
        P(&write);
    V(&mutex);
}
```

管程：高级语言实现的进程同步，进程们进入管程后，只有一个进程能够执行，在条件变量上排队。

#### 第八章 死锁

死锁产生的必要条件：<u>互斥、持有并等待、非抢占、循环等待</u>

解决死锁问题的方法：<u>死锁预防、死锁避免、死锁检测与恢复、鸵鸟算法（摆烂法，死锁出现后啥都不干）</u>

银行家算法

* 需要的数据结构：Available/Max/Allocation/Need；Work/Finish

* 算法思想：看看当前进程是不是可以，如果可以，尝试分配。

* 出题：(1) 给Available/Max/Allocation，判断是否为安全状态   (2) 从0时刻某个进程又增加要求，看看此时满不满足

例题：[(90条消息) 【操作系统】银行家算法的例题详解_只识闲人不识君的博客-CSDN博客_银行家算法例题](https://blog.csdn.net/weixin_46069678/article/details/106960153)

检测死锁发生后的解决措施：终止进程/资源抢占

#### 第九章

地址绑定的三个时期：编译时绑定、加载时绑定、执行时绑定

连续分配（如分段）：产生外部碎片；分页：产生内部碎片

TLB: Translation Look-aside Buffer

<font color="#2255ff">页表结构分为<u>层次页表、哈希页表、反向页表</u></font>

X86使用段页式内存管理：分段是必须的、而分页是可选的

#### 第十章

虚拟内存的两种实现形式：按需调页、按需调段

有效/无效位：为“无效”时，代表当前页无效、或者有效但在磁盘上

EAT：Effective Access Time计算方法：

* 当page fault发生的概率为$p$，内存访问时间为$m$时，

* $EAT = p \times page\_ fault\_ time + (1-p)\times m$

脏数据位（Dirty）：当目前页面有修改时，将脏数据位置为1，代表该页已经被修改

页面置换算法：FIFO/LRU/OPT。其中FIFO会产生Belady异常。

Page fault六个步骤：

* 检查进程内部表和页表，查看进程页引用是否合法。

* 若不合法，则结束进程；若合法但是valid为false，则进行swap

* 寻找一个空闲帧

* 调度磁盘操作，将需要的页面调入物理内存

* 修改PCB的有关内容，修改进程的内部表和页表

* 继续执行因page fault而中断的指令

系统颠簸产生的原因：进程增加多道程序程度导致引起更多页面错误导致CPU利用率大幅降低

解决系统颠簸的方法：使用<u>工作集</u>，是对程序局部的模拟

#### 第十一章

文件系统目录结构：<u>单层、两层、树型、无环图、图（General Graph，原文翻译为“通用图”，翻译得很烂）</u>

文件系统实现（分配方式）：连续分配、链式分配、索引分配
