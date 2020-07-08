
# 调度进程
调度进程的核心是__schedule()。
``` C
static void __sched notrace __schedule(bool preempt)
```
preempt表示是否抢占调度，true表示抢占调度，强制剥夺当前进程对处理器的使用权；false表示主动调度，让当前进程主动让出处理器。

抢占调度主要是为中断，异常等服务，包括preempt_schedule_common/preempt_schedule_notrace/preempt_schedule_irq。

__schedule（）并没有入参指定当前cpu，在函数中通过smp_processor_id()获取。

# 调度时机

+ 主动调度
+ 周期调度
+ 唤醒进程抢占
+ 创建新进程抢占
+ 内核抢占
+ 高精度调节时钟

## 内核抢占

### 主动调度
通过schedule()来申请主动调度，当前进程睡眠，等待调度的进程被主动调度

主要有三种主动调度方式：
+ 当进程认为自己暂时不需要占用处理器时，主动调用schedule(),让出cpu
+ 调用有条件重新调度函数cond_resched()。但是对于可抢占的内核，该函数为空。
+ 如果需要等待某个资源，例如互斥锁或者信号量，把进程设置为睡眠状态后，调用schedule()


### 主动内核抢占
preempt_enable

preempt_schedule

### 开启软中断内核抢占
软中断指的是中断下半文，
preempt_check_resched

preempt_schedule

### 释放自旋锁抢占

_cond_resched
