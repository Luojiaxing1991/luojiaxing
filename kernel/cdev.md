# cdev c文件

# cdev h文件


* root device register

root_device_register（）

# device.h
include/linux/device.h

vim ./bsg.c +489

# 另一种简单的读写-simple_read_from_buffer

之所以引入了这个，主要是file_operation里面的read元素包括一个loff_t的元素，用来标示已读数据到文件头的偏移，这个函数可以辅助处理该元素，不需要用户追加处理。
STATIC ssize_t (*read)(struct file *file, char __user *, size_t, loff_t *)

## 参考资料
https://blog.csdn.net/weixin_39821531/article/details/88556480

## 代码所在
kernel-4.9/fs/libfs.c

## simple_read_from_buffer


