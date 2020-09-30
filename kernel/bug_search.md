+ make W=1

# make W=1
通过编译告警刷patch是非常轻松的，同时也容易被接收。

可以主要关注下面这几类关键字：
1. set but not used      最简单的刷patch warning，同时也很多人刷
2. no previous prototype    有可能是缺少头文件，有可能是需要添加static，也比较简单修改
