# poll机制

## c文件
./fs/select.c

## 头文件
include/linux/poll.h

## 参考文档
https://www.jianshu.com/p/8cd91b71709a

https://www.cnblogs.com/amanlikethis/p/6915485.html

上面这两个文档有一个很让人误解的地方就是，首先澄清下。poll机制不需要配合waitqueue，只需要定义一个waitqueue给它即可，poll机制内部会使用这个waitqueue来实现一个睡眠唤醒机制。

## 一个poll机制的简单例子
```C
static DECLARE_WAIT_QUEUE_HEAD(my_waitq);  //休眠要挂的等待队列

static unsigned drv_poll(struct file *file, poll_table *wait)
{
  unsigned int mask = 0;
  poll_wait(file, &my_waitq, wait); // 不会立即休眠
  if (有数据) 
    mask |= POLLIN | POLLRDNORM; //poll机制会把mask作为 wait_queue的condition，所以，这个 有数据 的判断，就是我们需要设计的。
  return mask;
}

static void modify_condition()
{
  wake_up_poll(my_waitq, EPOLLIN);
}

```

poll机制内部会将my_waitq作为wait queue head，每次调用poll_wait就会在my_waitq里面新增wait entry。而且把drv_poll注册为wakeup时该entry默认执行的函数。这样每次wakeup事件发生，drv_poll会被调用，一旦condition符合，mask会被赋值，这样poll机制就成功找到了一个，并返回。
