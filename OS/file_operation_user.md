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

       The field fd contains a file descriptor for an open file.  If this
       field is negative, then the corresponding events field is ignored and
       the revents field returns zero.  (This provides an easy way of ignor‐
       ing a file descriptor for a single poll() call: simply negate the fd
       field.  Note, however, that this technique can't be used to ignore
       file descriptor 0.)

       The field events is an input parameter, a bit mask specifying the
       events the application is interested in for the file descriptor fd.
       This field may be specified as zero, in which case the only events
       that can be returned in revents are POLLHUP, POLLERR, and POLLNVAL
       (see below).

       The field revents is an output parameter, filled by the kernel with
       the events that actually occurred.  The bits returned in revents can
       include any of those specified in events, or one of the values
       POLLERR, POLLHUP, or POLLNVAL.  (These three bits are meaningless in
       the events field, and will be set in the revents field whenever the
       corresponding condition is true.)

       If none of the events requested (and no error) has occurred for any
       of the file descriptors, then poll() blocks until one of the events
       occurs.

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
    
### poll机制说明
https://www.cnblogs.com/amanlikethis/p/6915485.html

### poll 用户态使用 内核态回调处理
https://www.jianshu.com/p/8cd91b71709a

### event
事件类型events 可以为下列值：

POLLIN           有数据可读
POLLRDNORM 有普通数据可读，等效与POLLIN
POLLPRI         有紧迫数据可读
POLLOUT        写数据不会导致阻塞
POLLER          指定的文件描述符发生错误
POLLHUP        指定的文件描述符挂起事件
POLLNVAL      无效的请求，打不开指定的文件描述符

## close()
### manual of poll()
http://man7.org/linux/man-pages/man2/close.2.html

### 定义
int close(int fd)

       closes a file descriptor, so that it no longer refers to any
       file and may be reused.  Any record locks (see fcntl(2)) held on the
       file it was associated with, and owned by the process, are removed
       (regardless of the file descriptor that was used to obtain the lock).

       If fd is the last file descriptor referring to the underlying open
       file description (see open(2)), the resources associated with the
       open file description are freed; if the file descriptor was the last
       reference to a file which has been removed using unlink(2), the file
       is deleted.

## ioctl()
### manual of ioctl()
http://man7.org/linux/man-pages/man2/ioctl.2.html

### 定义
int ioctl(int fd, unsigned long request, ...);

       The ioctl() system call manipulates the underlying device parameters
       of special files.  In particular, many operating characteristics of
       character special files (e.g., terminals) may be controlled with
       ioctl() requests.  The argument fd must be an open file descriptor.

       The second argument is a device-dependent request code.  The third
       argument is an untyped pointer to memory.  It's traditionally char
       *argp (from the days before void * was valid C), and will be so named
       for this discussion.

       An ioctl() request has encoded in it whether the argument is an in
       parameter or out parameter, and the size of the argument argp in
       bytes.  Macros and defines used in specifying an ioctl() request are
       located in the file <sys/ioctl.h>.
       
 ## read()
 ### manual of read()
http://man7.org/linux/man-pages/man2/read.2.html

### 定义
ssize_t read(int fd, void *buf, size_t count)

       read() attempts to read up to count bytes from file descriptor fd
       into the buffer starting at buf.

       On files that support seeking, the read operation commences at the
       file offset, and the file offset is incremented by the number of
       bytes read.  If the file offset is at or past the end of file, no
       bytes are read, and read() returns zero.

       If count is zero, read() may detect the errors described below.  In
       the absence of any errors, or if read() does not check for errors, a
       read() with a count of 0 returns zero and has no other effects.

       According to POSIX.1, if count is greater than SSIZE_MAX, the result
       is implementation-defined; see NOTES for the upper limit on Linux.
