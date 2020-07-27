+ RCU简介
   + 作者与参考文档
   + 简介
   + 实现举例
   + RCU的API
      + rcu_read_lock/rcu_read_unlock
      + synchronize_rcu

# RCU简介

## 作者与参考文档
Linux内核中,RCU的文档位于在 /Documentation/RCU/。Paul E. McKenney是RCU源码的主要实现者，他把一些RCU的文章和论文链接整理如下：http://www2.rdrop.com/users/paulmck/RCU/

## 简介
RCU（Read-Copy Update）是数据同步的一种方式，其主要针对的数据对象是链表，目的是提高遍历读取数据的效率。在读取链表数据的时候使用RCU可以避免对链表进行耗时的加锁操作。例如，同一时间有多个线程同时读取该链表（通过RCU进行标识），这时候也可以允许一个线程对链表进行修改（但是修改的时候，不能用RCU，而需要加锁）。因此，RCU一般适用于需要频繁的读取数据，而相应修改数据并不多的情景。

## 实现举例
``` C
int xxx_read(struct A *a)
{
  rcu_read_lock(); //被rcu包括的这部分，rcu能保证a不会在执行过程中被其他抢占线程删除为空。同样是加锁的效果，但是rcu的开销比加锁低，效率高
  if (a)
    func(a);
  rcu_read_unlock();
}

int xxx_delete(struct A *a)
{
  synchronize_rcu(); //在释放a之前查看其他线程是否通过rcu锁定了a的读取或者拷贝操作，直到其他线程结束rcu_read_lock，不能释放a
  kfree(a);
}
```

## RCU的API

### 
