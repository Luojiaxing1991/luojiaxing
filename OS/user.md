# 字符设备操作函数

## 内核代码中的定义头文件
tools/include/nolibc/nolibc.h

## open()

### manual of open()
http://man7.org/linux/man-pages/man2/open.2.html

### 定义

对于已存的文件进行操作可以调用：
int open(const char *pathname, int flags);

对于不存在的文件进行open操作可以调用：
int open(const char *path, int flags, mode_t mode);

pathname： The open() system call opens the file specified by pathname.  If the
           specified file does not exist, it may optionally (if O_CREAT is
           specified in flags) be created by open().
           
flags: The argument flags must include one of the following access modes:
       O_RDONLY, O_WRONLY, or O_RDWR.  These request opening the file read-
       only, write-only, or read/write, respectively.

       In addition, zero or more file creation flags and file status flags
       can be bitwise-or'd in flags.  The file creation flags are O_CLOEXEC,
       O_CREAT, O_DIRECTORY, O_EXCL, O_NOCTTY, O_NOFOLLOW, O_TMPFILE, and
       O_TRUNC.  The file status flags are all of the remaining flags listed
       below.  The distinction between these two groups of flags is that the
       file creation flags affect the semantics of the open operation
       itself, while the file status flags affect the semantics of
       subsequent I/O operations.  The file status flags can be retrieved
       and (in some cases) modified; see fcntl(2) for details.

## poll()

### manual of poll()
http://man7.org/linux/man-pages/man2/poll.2.html

### 定义
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

fds：  The set of file descriptors to be monitored is specified in the fds
       argument, which is an array of structures of the following form:

           struct pollfd {
               int   fd;         /* file descriptor */
               short events;     /* requested events */
               short revents;    /* returned events */
           };

nfds:  The caller should specify the number of items in the fds array in
       nfds.
       
timeout:The timeout argument specifies the number of milliseconds that poll()
       should block waiting for a file descriptor to become ready.  The call
       will block until either:

       *  a file descriptor becomes ready;

       *  the call is interrupted by a signal handler; or

       *  the timeout expires.

       Note that the timeout interval will be rounded up to the system clock
       granularity, and kernel scheduling delays mean that the blocking
       interval may overrun by a small amount.  Specifying a negative value
       in timeout means an infinite timeout.  Specifying a timeout of zero
       causes poll() to return immediately, even if no file descriptors are
       ready.
       
 ## close()
 


