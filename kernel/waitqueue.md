# waitqueue介绍
  wq主要用于同一个进程下的线程间进行信息同步。
  
  首先解释下为什么是个queue，因为如果你init一个wq，那么这个进程下的所有线程都可以往这个wq里面去新增一个wq entry。当然这个新增操作对于每一个线程都是透明的。每个线程通过调用wait_event()函数，就可以在wq里面创建自己的entry。笔者认为，通过这种汇聚可以减少资源的占用，但是通过wakeup去唤醒wq时，会轮询下面所有entry，这样相对应就会造成性能下降。
  
  因此，当一个线程在使用wq的时候，它对应了里面的一个entry（主要保存一些线程恢复相关的信息）。在调用wait_event（）函数时，线程会指定一种condition，如果在初始化的时候condition就满足，那就直接返回继续往下执行线程；否则，调用schedule()将本线程睡眠，同时调度其他线程执行。如果wakeup事件/timeout/signal到来,wq会检查condition是否满足，如果满足，会唤醒当前entry对应的线程继续执行；对于condition未满足，则继续睡眠。当然，对于timeout/signal的情况，无论condition是什么情况，线程也会被唤醒。
  
  对于wait_event的返回值，由于wait_event有很多变种，这个需要根据不同变种进行分析。举例： wait_event_interruptible_timeout，这个函数的返回值分为四种：
  0 if the @condition evaluated to %false after the @timeout elapsed,
  1 if the @condition evaluated to %true after the @timeout elapsed,  --这种情况就对应，condition满足后忘记/来不及发wakeup，而直接timeout了
  the remaining jiffies (at least 1) if the @condition evaluated to %true before the @timeout elapsed,
  -%ERESTARTSYS if it was interrupted by a signal  --该进程接受到signal，必须进行唤醒
  

# A simple example of wq
![Image text](https://github.com/Luojiaxing1991/picture/blob/master/simple_example_of_waitqueue.PNG)

# 内核文件

头文件：
./include/linux/wait.h

C文件：
./kernel/sched/wait.c
