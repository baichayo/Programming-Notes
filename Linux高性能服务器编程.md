<h1>Linux高性能服务器编程</h1>

## tcpdump

`tcpdump` 是一个在命令行下使用的网络抓包工具，用于捕获和分析网络数据包。它可以帮助你监控网络流量、调试网络问题以及分析网络通信。

```text
tcpdump options proto dir type
```

![tcpdump](D:\typora\md文件\imgs\tcpdump.png "你是猪吗")

SYNOPSIS
       tcpdump [ -AbdDefhHIJKlLnNOpqStuUvxX# ] [ -B buffer_size ]
               [ -c count ]
               [ -C file_size ] [ -G rotate_seconds ] [ -F file ]
               [ -i interface ] [ -j tstamp_type ] [ -m module ] [ -M secret ]
               [ --number ] [ -Q in|out|inout ]
               [ -r file ] [ -V file ] [ -s snaplen ] [ -T type ] [ -w file ]
               [ -W filecount ]
               [ -E spi@ipaddr algo:secret,...  ]
               [ -y datalinktype ] [ -z postrotate-command ] [ -Z user ]
               [ --time-stamp-precision=tstamp_precision ]
               [ --immediate-mode ] [ --version ]
               [ expression ]

- `-A`: 显示数据包的 ASCII 内容。这对于查看数据包中的实际数据非常有用。
- `-c count`: 指定要捕获的数据包数量。一旦捕获的数据包达到指定数量，`tcpdump` 将自动停止。
- `-e`：同时获取以太网标头
- `-i interface`: 指定要监听的网络接口。可以通过接口名来指定，例如 `-i eth0`。
- `-n`: 禁用 DNS 解析。使用此选项将在输出中显示 IP 地址而不是主机名。
- `-q`: 简化输出。使用此选项可以减少输出的详细程度。
- `-r file`: 从指定的文件中读取数据包。这对于离线分析捕获的数据包非常有用。
- `-s snaplen`: 指定捕获的数据包的最大大小。默认情况下，`tcpdump` 捕获所有数据，但是你可以限制捕获的数据包大小。
- `-t` 选项用于在输出中显示时间戳
- `-v`: 显示详细信息。使用此选项可以输出更多关于数据包的详细信息，例如源和目标 IP 地址、端口等。
- `-w file`: 将捕获的数据包保存到指定文件中。捕获的数据包将以 pcap 格式保存，后续可以在其他工具中分析。
- `-X`: 显示数据包的十六进制和 ASCII 内容。这对于深入分析数据包内容很有帮助。



## ps

`ps` 是一个用于显示当前系统中运行进程信息的命令。

1. `-A`：显示所有进程，包括其他用户的进程。
2. `-u`：显示属于指定用户的进程信息。
3. `-x`：显示没有控制终端的进程。
4. `-e`：显示所有进程。
5. `-f`：显示完整的进程信息，包括命令行参数和其他详细信息。
6. `-l`：显示长格式输出，包括更多的列信息。
7. `-o`：自定义输出格式，可以通过逗号分隔的选项指定要显示的列。
8. `-p`：显示指定进程ID的进程信息。
9. `-t`：显示指定终端的进程信息。
10. `--sort`：按指定的列进行排序输出。
11. `--forest`：以树状形式显示进程层级关系。
12. `--pid`：显示指定进程ID的信息。
13. `--ppid`：显示指定父进程ID的进程信息。
14. `--user`：显示指定用户的进程信息。
15. `--group`：显示指定用户组的进程信息



## netstat

`netstat` 是一个用于查看网络连接、路由表和网络统计信息的命令行工具。它可以帮助你了解系统的网络活动情况。



- `-a`: 显示所有的网络连接，包括监听和已建立的连接。
- `-t`: 仅显示 TCP 连接信息。
- `-u`: 仅显示 UDP 连接信息。
- `-n`: 显示数字格式的 IP 地址和端口号，避免进行主机名和服务名解析。
- `-l`: 显示所有正在监听的端口及其相关信息。
- `-p`: 显示与每个网络连接关联的 PID（进程标识符）和程序名。
- `-r`: 显示系统的路由表，包括目标网络、网关、接口等信息。
- `-s`: 显示网络相关的统计信息，如收发的数据包、错误等。
- `-c`: 持续显示输出，每隔一段时间刷新一次。
- `-W`: 指定显示的宽度，可用于格式化输出。
- `-i`: 显示网络接口的统计信息，如传输、丢包等。



## firewall-cmd

### 基本语法

```text
firewall-cmd --zone=zone-name --add-service=service-name --permanent
````

### 命令参数

- --zone：指定要添加服务的区域名称。
- --add-service：指定要添加的服务名称。
- --permanent：指定该规则永久生效。

除此之外，还有其他可选参数：

- --list-all：列出所有规则。
- --reload：重新加载防火墙规则。
- --permanent：将规则保存到永久配置中，以便系统重启后仍然有效。
- --delete-service：删除服务。



## chapter_05 Linux网络编程基础 API

### 地址信息函数

```c
#include <sys/socket.h>

/*
sockfd	套接字文件描述符
addr	将获取的地址存于 addr
addrlen	用于存储 addr 长度，如果 addr 的大小大于 addlen ，则地址将被截断
*/

//成功返回 0，失败返回 -1，并设置 errno
int getsockname(int sockfd, struct sockaddr *addr, socklen_t *addrlen);

int getpeername(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

实验结果：

- 服务器：

![image-20230808143309606](D:\typora\md文件\imgs\LinuxServer_chapter_05_01.png)

- 客户端

![image-20230808143618251](D:\typora\md文件\imgs\LinuxServer_chapter_05_02.png)

结果分析：

可以看到，不管服务器还是客户端，地址都是保存的对方的 `IP + Port`，但有一点有点疑惑，服务器的 `serv_addr` 的 端口竟然是 7433，而不是我们指定的 2333 。





## chapter_06 高级 I/O 函数

### sendfile

```c
#include <sys/sendfile.h>

/*
out_fd	文件描述符，必须是套接字（Linux 2.6.33 开始，它可以是任何文件）
in_fd	不能是 socket 或 管道
offset	偏移量
count	是在文件描述符之间复制的字节数，可以用 fstat 获取文件大小
*/

//成功返回写入 out_fd 的字节数，失败返回 -1，并设置 errno
ssize_t sendfile(int out_fd, inte, off_t *offset, size_t count);
```


### splice

`splice()` 在两个文件描述符之间移动数据，而不在内核地址空间和用户地址空间之间进行复制。 它将**最多** `len` 个字节的数据从文件描述符 `fd_in` 传输到文件描述符 `fd_out`，**其中文件描述符之一必须引用管道**。

```c
#define _GNU_SOURCE         /* See feature_test_macros(7) */
#include <fcntl.h>

//成功返回移动字节的数量，EOF返回 0，失败返回 -1
ssize_t splice(int fd_in, loff_t *off_in, int fd_out,
                      loff_t *off_out, size_t len, unsigned int flags);
```

<table>
    <caption> <code>splice</code> 的 <code>flags</code> 参数</caption>
    <tr>
        <th>常用值</th> <th>含义</th>
    </tr>
    <tr>
        <td>SPLICE_F_MOVE</td> <td>如果合适的话，按整页内存移动数据。不过内核 2.6.21 后没任何效果</td>
    </tr>
    <tr>
        <td>SPLICE_F_NONBLOCK</td> <td>非阻塞的 splice 操作，实际效果收文件描述符本身阻塞状态影响</td>
    </tr>
    <tr>
        <td>SPLICE_F_MORE</td> <td>给内核的一个提示：后续的 splice 调用将读取更多数据</td>
    </tr>
    <tr>
        <td>SPLICE_F_GIFT</td> <td>对 splice 没有效果</td>
    </tr>
</table>


使用体验：对偏移量不太理解，在将管道的数据写到文件中时，都设置了 `NULL` ，不过数据并没有被覆盖，而是以 `append` 的方式写到文件了。

- 不指定偏移量，即 `off_out` 为 `NULL`，则：
    - `off_out` 指向保存文件偏移量的变量
    - 在完成操作后，会更新文件偏移量

- 如果指定偏移量，则：
	- 从指定的偏移量开始写数据
	- 完成操作后，文件偏移量不会被改变
	- 返回最后一个字节之后的字节的偏移量（左闭右开啦）

### tee

`tee` 用于在两个管道文件描述符之间复制数据，它**不消耗数据**。`fd_in`和`fd_out`都必须是管道文件描述符
`splice`用于移动数据，会消耗数据。


```c
#include <fcntl.h>

/*
len		貌似也是最大字节数
*/

//成功返回复制的字节数，返回 0 说明没有复制任何数据，失败返回 -1 并设置 errno
ssize_t tee(int fd_in, int fd_out, size_t len, unsigned int flags);
```



### fcntl

file control 函数提供对文件描述符的各种控制操作。

```c
#include <unistd.h>
#include <fcntl.h>

//成功时返回值如下表，失败返回 -1，并设置 errno
int fcntl(int fd, int cmd, ...);
```



<table border = "1">
    <caption><code>fcntl</code>支持的常用操作及其参数</caption>
    <tr>
        <th>操作分类</th> <th>操作</th> <th>含义</th> <th>第三个参数类型</th> <th>成功时的返回值</th>
    </tr>
    <tr>
        <td rowspan = "2">复制文件描述符</td>
        <td>F_DUPFD</td>
        <td>创建一个新的文件描述符，其值大于或等于 arg</td>
        <td>long</td> <td>新创建的文件描述符的值</td>
    </tr>
    <tr>
        <td>F_DUPFD_CLOEXEC</td>
        <td>与 F_DUPFD 类似，不过在创建文件描述符的同时，设置其 close-on-exec 标志</td>
        <td>long</td> <td>新创建的文件描述符的值</td>
    </tr>
    <tr>
        <td rowspan = "2">获取和设置文件描述符的标志</td>
        <td>F_GETFD</td>
        <td>获取 fd 的标志，比如 close-on-exec 标志</td>
        <td>无</td> <td>fd 的标志</td>
    </tr>
    <tr>
        <td>F_SETFD</td>
        <td>设置 fd 的标志</td>
        <td>long</td> <td>0</td>
    </tr>
    <tr>
        <td rowspan = "2">获取和设置文件描述符的状态标志</td>
        <td>F_GETFL</td>
        <td>获取 fd 的状态标志，这些标志包括可由 open 系统调用 设置的标志（O_APPEND O_CREAT等）和访问模式（O_RDONLY等）</td>
        <td>void</td> <td>fd 的状态标志</td>
    </tr>
    <tr>
        <td>F_SETFL</td>
        <td>设置 fd 的状态标志，但部分标志是不能被修改的（比如访问模式标志）</td>
        <td>long</td> <td>0</td>
    </tr>
    <tr>
        <td rowspan = "4">管理信号</td> <td>F_GETOWN</td>
        <td>获取 SIGIO 和 SIGURG 信号的宿主进程的 PID 或进程组的组 PID</td>
        <td>无</td> <td>信号的宿主进程的 PID 或进程组的组 PID</td>
    </tr>
    <tr>
        <td>F_SETOWN</td>
        <td>设定 SIGIO 和 SIGURG 信号的宿主进程的 PID 或进程组的组 PID</td>
        <td>long</td> <td>0</td>
    </tr>
    <tr>
        <td>F_GETSIG</td>
        <td>获取当应用程序被通知 fd 刻度或可写时，是哪个信号通知该事件的</td>
        <td>无</td> <td>信号值，0 表示 SIGIO</td>
    </tr>
    <tr>
        <td>F_SETSIG</td>
        <td>设置当 fd 可读或可写时，系统应该触发哪个信号来通知应用程序</td>
        <td>long</td> <td>0</td>
    </tr>
    <tr>
        <td rowspan = "2">操作管道容量</td>
        <td>F_SETPIPE_SZ</td>
        <td>设置由<code>fd</code> 指定的管道的容量</td>
        <td>long</td>
        <td>0</td>
    </tr>
    <tr>
        <td>F_GETPIPE_SZ</td>
        <td>获取由 <code>fd</code> 指定的管道的容量</td>
        <td>无</td>
        <td>管道容量</td>
    </tr>
</table>





## chapter_07 Linux 服务器程序规范

很好，鸽了









## chapter_08 高性能服务器程序框架

talk is cheap, show me code.



### I/O 模型

- 阻塞的文件描述符称为 阻塞 I/O

- 非阻塞的文件描述符称为 非阻塞 I/O

- 针对阻塞 I/O 执行的系统调用可能因为无法立即完成而被操作系统挂起，直到等待的事件发生为止

- 针对非阻塞 I/O 执行的系统调用总是立即返回，不管事件是否已经发生。如果事件没有立即发生，这些系统调用就返回 -1 ，和出错的情况一样。因此我们必须根据 errno 来区分这两种情况。

    

- 显然（我也不知道哪显然），只有在事件已经发生的情况下操作非阻塞 I/O，才能提高程序的效率。因此，非阻塞 I/O 通常要和其他 I/O 通知机制一起使用，比如 I/O 复用和 SIGNO 信号。

- I/O 复用函数本身是阻塞的，它们能提高程序效率的原因在于它们具有同时监听多个 I/O 事件的能力。（结合这两点，如果是非阻塞的，则必须不断 while 循环，但有了阻塞的 I/O 复用函数，能够确保事件已经发生，则可以避免使用 while ，当然有些情况还是躲不掉）

    

- SIGIO 信号也可以用来报告 I/O 事件。可以为一个目标文件描述符指定宿主进程，那么被指定的宿主进程将捕获到 SIGIO 信号。


- 从理论上来说，阻塞 I/O 、I/O 复用和信号驱动 I/O 都是同步 I/O 模型。因为在这三种 I/O 模型中， I/O 的读写操作，都是在 I/O 事件发生之后，**由应用程序完成**。而 POSIX 规范所定义的异步 I/O 模型则不同，对异步 I/O 而言，用户可以直接对 I/O 执行读写操作，并立即返回结果，**读写操作由内核接管**。可以这样认为，同步 I/O 向应用程序通知的是 I/O 就绪事件，而异步 I/O 向应用程序通知的是 I/O 完成时间。

    

<table>
	<caption>I/O 模型对比</caption>
	<tr>
		<th>I/O 模型</th> <th>读写操作和阻塞阶段</th>
	</tr>
	<tr>
		<td>阻塞 I/O</td> <td>程序阻塞于读写函数</td>
	</tr>
	<tr>
		<td>I/O 复用</td> <td>程序阻塞于 I/O 复用系统调用，但可同时监听多个 I/O 事件。对 I/O 本身的读写操作是非阻塞的</td>
	</tr>
	<tr>
		<td>SIGIO 信号</td> <td>信号触发读写就绪事件，用户程序执行读写操作。程序没有阻塞阶段</td>
	</tr>
	<tr>
		<td>异步 I/O</td> <td>内核执行读写操作并触发读写完成时间。程序没有阻塞阶段</td>
	</tr>
</table>



### 两种高效的事件处理模式

服务器通常需要处理三类事件：I/O 事件，信号及定时事件。

同步 I/O 模型通常用于实现 Reactor 模式，异步 I/O 模型则用于实现 Proactor 模式

#### Reactor 模式

主线程只负责监听文件描述符上是否有事件发生，有的话立即将该事件通知工作线程。读写数据，接受新的连接，以及处理客户请求均在工作线程中完成。



使用同步 I/O 模型（以 epoll_wait 为例）实现的 Reactor 模式的工作流程是：

- 主线程往 epoll 内核时间表中注册 socket 上的读就绪事件。
- 主线程调用 epoll_wait 等待 socket 上有数据可读
- 当 socket 上有数据可读时， epoll_wait 通知主线程。主线程则将 socket 可读事件放入请求队列。
- 睡眠在请求队列上的某个工作线程被唤醒，它从 socket 读取数据，并处理客户请求，然后往 epoll 内核事件表中注册该 socket 上的写就绪事件。
- 主线程调用 epoll_wait 等待 socket 可写。
- 当 socket 可写时， epoll_wait 通知主线程。主线程将 socket 可写事件放入请求队列。
- 睡眠在请求队列上的某个工作线程被唤醒，它往 socket 上写入服务器处理客户请求的结果。



小结：

- 主线程监听着所有连接的事件，工作线程不负责监听事件。
- 所有线程共用一个 epfd（额，好像没那么简单？） 。
- 疑惑的是，“当 socket 可写时， epoll_wait 通知主线程” 是什么意思。什么叫 当 socket 可写时。



#### Proactor 模式

Proactor 模式将所有 I/O 操作都交给主线程和内核来处理，工作线程仅仅负责业务逻辑。



使用异步 I/O 模型（以 aio_read 和 aio_write 为例）实现的 Proactor 模式的工作流程是：

- 主线程调用 aio_read 函数向内核注册 socket 上的读完成事件，并告诉内核用户读缓冲区的位置，以及读操作完成时如何通知应用程序（以信号为例）。
- 主线程继续处理其他逻辑。
- 当 socket 上的数据被读入用户缓冲区后，内核将应用程序发送一个信号，以通知应用程序数据已经可用。
- 应用程序预先定义好的信号处理函数选择一个工作线程来处理客户请求。工作线程处理完客户请求之后，调用 aio_write 函数向内核注册 socket 上的写完成事件，并告诉内核用户写缓冲区的位置，以及写操作完成时如何通知应用程序（以信号为例）。
- 主线程继续处理其他逻辑。
- 当用户缓冲区的数据被写入 socket 之后，内核将向应用程序发送一个信号，以通知应用程序数据已经发送完毕。
- 应用程序预先定义好的信号处理函数选择一个工作线程来做善后处理，比如决定是否关闭 socket 。



使用同步 I/O 模型（以 epoll_wait 为例）实现的 Proactor 模式的工作流程是：

- 主线程往 epoll 内核事件表中注册 socket 上的读就绪事件。
- 主线程调用 epoll_wait 等待 socket 上有数据可读
- 当 socket 上有数据可读时， epoll_wait 通知主线程。主线程从 socket 循环读取数据，直到没有更多数据可读，然后将读取到的数据封装成一个请求对象并插入请求队列。
- 睡眠在请求队列上的某个工作线程被唤醒，它获得请求对象并处理客户请求，然后往 epoll 内核事件表中注册 socket 上的写就绪事件。
- 主线程调用 epoll_wait 等待 socket 可写
- 当 socket 可写时， epoll_wait 通知主线程。主线程往 socket 上写入服务器处理客户请求的结果。





### 两种高效的并发模式

并发编程的目的是 让程序 “同时”执行多个任务。

并发模式是指 I/O 处理单元和多个逻辑单元之间协调完成任务的方法。

服务器主要有两种并发编程模式：半同步/半异步（half-sync/half-async）模式和领导者/追随者（Leader/Followers）模式



#### 半同步/半异步模式

- 在 I/O 模型中，同步和异步区分的是内核向应用程序通知的是何种 I/O 事件（就绪事件还是完成事件），以及该由谁来完成 I/O 读写（应用程序还是内核）。
- 在并发模式中，同步指的是程序完全按照代码序列的顺序执行；异步指的是程序的执行需要由系统事件来驱动。常见的系统事件包括终端、信号等。

按照同步运行的线程称为同步线程，按照异步方式运行的线程称为异步线程。



半同步/半异步模式中，同步线程用于处理客户逻辑；异步线程用于处理 I/O 事件。

异步线程监听到客户请求后，将其封装成请求对象并插入请求队列中。请求队列将通知某个工作在同步模式的工作线程来读取并处理该请求对象。



`http_conn` 是用半同步/半反应堆模式实现的简单 Web 服务器的代码

半同步/半反应堆模式存在如下缺点：

- 主线程和工作线程共享请求队列。主线程往请求队列中添加任务，或者工作线程从请求队列中取出任务，都需要对请求队列加锁保护，从而白白耗费 CPU 事件。
- 每个工作线程在同一时间只能处理一个客户请求。如果客户请求数量较多，而工作线程较少，则请求队列中将堆积很多任务对象，客户端的响应速度将越来越慢。



#### 领导者/追随者模式

领导者/追随者模式是多个工作线程轮流获得事件源集合，轮流监听、分发并处理事件的一种模式。在任意时间点，程序都仅有一个领导者线程，它负责监听 I/O 事件。



没得代码，没得理解






## chapter_09 I/O 复用

I/O 复用使得程序能同时监听多个文件描述符，这对提高程序的性能至关重要。通常，网络程序在下列情况下需要使用 I/O 复用技术：

- 客户端程序要同时处理多个 socket ，如 非阻塞 connect 技术 （虽然没看懂）
- 客户端程序要同时处理用户输入和网络连接。（poll_clnt.c）
- **TCP 服务器要同时处理监听 socket 和 连接 socket 。**
- 服务器要同时处理 TCP 请求 和 UDP 请求。
- 服务器要同时监听多个端口，或者处理多种服务。


### select 系统调用

```c
#include <sys/select.h>

//成功返回三个返回的描述符集中包含的文件描述符的数量
//超时返回 0，失败返回 -1，并设置 errno
int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, timeval *timeout);
```

使用 `select`：
- 定义监听事件集合及它们的拷贝
- 调用宏 `FD_ZERO()` 初始化集合（将所有位置为 0）
- 调用宏 `FD_SET()` 将需监听的文件描述符注册到对应事件集合（对应位置为 1）
- 调用 `select()` 开始监听
- 轮询，调用 `FD_ISSET()` 判断是否发生相应事件


#### 文件描述符就绪条件

哪些情况下文件描述符可以被认为是可读、可写或者出现异常，对于 `select` 的使用非常关键。在网络编程中，

- 下列情况下 socket 可读：
	- socket 内科接收缓存区中的字节数 >= 其低水位标记 `SO_RCVLOWAT` 。此时读操作返回的字节数大于 0 。
	- socket 通信的对方关闭连接。此时对该 socket 的读操作返回 0 。
	- 监听 socket 上有新的连接请求
	- socket 上有未处理的错误。我们可以使用 `getsockopt` 来读取和清除该错误。

- 下列情况下 socket 可写：
	- socket 内核发送缓存区中的可用字节数 >= 其低水位标记 `SO_SNDLOWAT` 。
	- socket 的写操作被关闭。此时执行写操作将触发 `SIGPIPE` 信号。
	- socket 使用非阻塞 `connect` 连接成功或者失败（超时）之后
	- socket 上有未处理的错误。
	

网络程序中，socket 能处理的异常情况只有一种：socket 上接收到带外数据



### poll 系统调用

```c
#include <poll.h>

//成功返回三个返回的描述符集中包含的文件描述符的数量
//超时返回 0，失败返回 -1，并设置 errno
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

```c
struct pollfd
{
	int fd;			//文件描述符
    short events;	//注册的事件
    short revents; 	//用于内核填充发生的事件
}
```


使用 `poll`:
- 定义 `struct pollfd` 集合
- 将文件描述符注册到集合
- 调用 `poll()` 监听
- 轮询，`pollfds[i].revents & POLLIN` 来判断是否发生相应事件



### epoll 系列系统调用

```c
#include <sys.epoll.h>

//成功返回非负文件描述符，失败返回 -1，并设 errno
int epoll_create(int size);

//成功返回 0，失败返回 -1，并适当设置 errno
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

//成功返回就绪文件描述符数量，超时返回 0，失败返回 -1，并适当设置 errno
int epoll_wait(int epfd, struct epoll_event *events,
               	int maxevents, int timeout);
```



使用 `epoll` ：
- 调用 `epoll_create()` 创建 `epoll` 文件描述符（`epfd`）
- 创建 `struct epoll_event` ，绑定 文件描述符和监听事项
- 调用 `epoll_ctl()` 向 `epfd` 注册事件
- 调用 `epoll_wait()` 监听事件
- `ep_events[i].data.fd == serv_sock` 和 `ep_events[i].events & EPOLLIN` 判断是否发生相应事件

#### LT 和 ET 模式

`select` 和 `poll` 都是条件触发（Level Trigger）模式， `epoll` 默认条件触发，但 `epoll` 还有另一种模式——边缘触发（Edge Trigger）

在 ET 模式中，文件描述符必须设置为非阻塞：
- `serv_sock` 设置为非阻塞：`accept()` 函数需立即返回（额，虽然我想象不到会阻塞的情况）
- `clnt_sock` 设置为非阻塞：由于事件只会触发一次，因此我们需要一次性读完 TCP 缓冲中的所有数据，那么就需要用 `while` 循环，当读完时应当返回，因此需要设置为非阻塞，并且与 `errno` 配合 `break` 循环。

#### EPOLLONESHOT 事件

对于注册了 `EPOLLONESHOT` 事件的文件描述符，操作系统最多触发其上注册的一个可读、可写或者异常事件，且只触发一次。
该事件能进一步减少可读、可写和异常等事件被触发的次数。

具体用法请参考 `epolloneshot.c`



### I/O 复用的高级应用一：非阻塞 `connect`

`connect` 出错时的一种 `errno` 值：`EINPROGRESS` 。这种错误发生在对非阻塞的 socket 调用 `connect` ，而连接又没有建立时。

在这种情况下，可以调用 `select` 、 `poll` 等函数来监听这个连接失败的 socket 上的 **可写**事件。当 `select` 、 `poll` 等函数返回后，再利用 `getsockopt` 来读取错误码并清楚该 socket 上的错误。

如果错误码是 0 ，表示连接成功建立，否则连接失败

代码请看……鸽了


### I/O 复用的高级应用二：聊天室程序

客户端：
- 从标准输入终端读入用户数据，并将用户数据发送至服务器。
- 往标准输出端打印服务器发送给它的数据。

服务器：
- 接收客户数据
- 将客户数据发送给每一个登录到该服务器上的客户端。

代码请看：`poll_clnt.c` 和 `poll_serv.c`

### I/O 复用的高级应用三：同时处理 TCP 和 UDP 服务

服务器可以同时监听多个端口，不过对于同一种套接字，套接字和端口一对一。

代码：`echo_multiport.c`

### 超级服务 xinetd

咯咯咯



## chapter_10 信号

信号产生条件：
- 对于前台进程，用户可以通过输入特殊的终端字符来给它发送信号。
- 系统异常。比如浮点异常和非法内存段访问。
- 系统状态变化。比如 `alarm` 定时器到期将引起 `SIGALRM` 信号。
- 运行 `kill` 命令或调用 `kill` 函数。


### 发送信号

```c
#include <sys/types.h>
#include <signal.h>

//成功返回 0 ，失败返回 -1 ，并设置 errno
int kill(pid_t pid, int sig);
```

`pid` 参数：
- pid > 0 ：信号发送给 PID 为 pid 的进程
- pid = 0 ：信号发送给本进程组内的其他进程
- pid = -1 ：信号发送给除 init 进程外的所有进程，但发送者需要拥有对目标进程发送信号的权限
- pid < -1 ：信号发送给组 ID 为` abs(pid)` 的进程组中的所有成员

`errno` 值：
- `EINVAL` ：无效的信号
- `EPERM` ：该进程没有权限发送信号给任何一个目标进程
- `ESRCH` ：目标进程或进程组不存在

### 信号处理方式

#### 系统信号处理函数

```c
#include <bits/signum.h>

#define SIG_DFL ((__sighandler_t) 0)
#define SIG_IGN ((__sighandler_t) 1)
```

`SIG_IGN` 表示忽略目标信号，`SIG_DFL` 表示使用信号的默认处理方式。

默认处理方式：
- `Term` ：结束进程
- `Ign`   ：忽略信号
- `Core` ：结束进程并生成核心转储文件
- `Stop` ：暂停进程
- `Cont` ：继续进程


#### 用户自定义信号处理函数

```c
#include <signal.h>

typedef void (* __sighandler_t)(int);
```


### 统一事件源

信号是一种异步函数：信号处理函数和程序的主循环是两条不同的执行路线。

为了避免一些竞态条件，**信号在处理期间，系统不会再次触发它**。

因此信号处理函数需要尽可能快地执行完毕，以确保该信号不被屏蔽太久。



典型处理方式是，当信号处理函数被触发时，它只是简单地将信号值传递给主循环，交给主循环处理信号。

方法为：信号处理函数将信号值写入管道，主循环用 I/O 复用系统监听管道的可读事件。

如此一来，信号事件就能和其他 I/O 事件一样被处理，即 统一事件源。






## chapter_11 定时器

啧，鸽了





## chapter_12 高性能 I/O 框架库 Libevent

额，以后再了解。




## chapter_13 多进程编程

**IPC(Inter-Process Communication)**:

- Pipe: 管道是一种最简单的IPC方式，用于在父进程和子进程之间传递数据。管道可以是匿名的，也可以是有名字的。匿名管道只适用于有亲缘关系的进程，而命名管道（FIFO）可以在无关进程之间使用。
- Message Queue: 消息队列是一种存储在内核中的数据结构，用于在不同进程之间传递消息。
- Semaphore: 信号量是一种计数器，用于控制多个进程对共享资源的访问。通过对信号量进行P（等待）和V（释放）操作，可以协调进程的访问，避免竞争条件。
- Shared Memory: 共享内存是最高效的 IPC 机制，因为它不涉及进程之间的任何数据传输。共享内存通常和其他进程间通信方式一起使用。
- Socket: 套接字可以用于进程间通信，也可以用于不同计算机之间的通信。`socketpair` 是全双工通信。
- Signal: 信号是一种异步通知机制，用于在进程之间传递异步事件
- RPC(Remote Procedure Call): RPC是一种高级的IPC机制，允许在不同的计算机上调用远程的过程或函数，就像调用本地的过程一样

### 共享内存

#### shmget

用于创建或访问共享内存

```c
#include <sys/ipc.h>
#include <sys/shm.h>

/*
key		用来表示一个全局唯一的共享内存
size	指定共享内存的大小
shmflg	一组标志，用于指定共享内存段的创建和访问权限
*/

//成功返回内存标识符，错误返回 -1 ，并设 errno
int shmget(key_t key, size_t size, int shmflg);
```

#### shmat & shmdt

共享内存被创建/获取后，我们不能立即访问它，而是需要先将他关联到进程的地址空间中。

使用完共享内存后，我们也需要将它从进程地址空间中分离。

```c
#include <sys/types.h>
#include <sys/shm.h>

/*
shmid		shmget() 调用返回的共享内存标识符
shmaddr		指定将共享内存关联到进程的哪块地址空间，一般为 NULL ，交由操作系统选择
shmflg		附加标志
*/

//成功返回 附加内存段的地址，出错返回 -1 ，并设 errno
void *shmat(int shmid, const void *shmaddr, int shmflg);


//成功返回 0 ，失败返回 -1 ，并设 errno
int shmdt(const void *shmaddr);
```



#### shmctl

用于控制共享内存段的操作。

它可以用于获取或设置共享内存段的状态、删除共享内存段以及执行其他控制操作

```c
#include <sys/ipc.h>
#include <sys/shm.h>

/*
shmid	shmget() 调用返回的共享内存标识符
cmd		要执行的操作
buf		用于存储共享内存段的信息
*/

//成功返回 0 或其他值，失败返回 -1 ，并设 errno
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

| 命令       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `IPC_STAT` | 获取共享内存段的状态信息，包括权限、大小、拥有者等。         |
| `IPC_SET`  | 设置共享内存段的状态信息，通常用于修改权限。                 |
| `IPC_RMID` | 删除共享内存段。会使连接到共享内存的所有进程分离并销毁共享内存。 |
| ...        | ...                                                          |

#### 共享内存的 POSIX 方法

```c
#include <sys/mman.h>
#include <sys/stat.h>        /* For mode constants */
#include <fcntl.h>           /* For O_* constants */

/*
name	共享内存对象的名字.如 "/my_shm"
oflag	指定创建方式。 O_RDONLY O_RDWR O_CREAT O_EXCEL O_TRUNC
mode	创建共享内存对象时的权限。通常是 0666
*/

//成功返回 非负文件描述符，失败返回 -1
int shm_open(const char *name, int oflag, mode_t mode);


//成功返回 0 ，失败返回 -1
int shm_unlink(const char *name);

//Link with -lrt.
```



组合技

```c
int shmfd = shm_open("/my_shm", O_CREAT | O_RDWR, 0666);
ftruncate(shmfd, XXX_SIZE); //调整文件大小
char *share_mem = (char *)mmap(NULL, XXX_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, shmfd, 0); //将文件或其他对象映射到进程的地址空间，用于进程间共享数据
close(shmfd);
```



#### mmap

用于将文件或其他对象映射到进程的地址空间，从而允许直接读写内存中的数据，而无需进行显式的读写操作。这种内存映射的方式通常可以提高读写效率，同时也可以用于一些特定的用途，如共享内存通信。

```c
#include <sys/mman.h>

/*
addr	映射的首地址，通常为 NULL
length	映射的长度
prot	映射的保护模式，用于指定内存的访问权限
flags	映射的标志，用于指定映射方式和行为
fd		要映射的文件描述符
offset	要映射的文件的偏移量
*/

//成功返回映射区域的指针，失败返回 -1 ，并设置 errno
void *mmap(void *addr, size_t length, int prot, int flags,
                  int fd, off_t offset);
int munmap(void *addr, size_t length);
```





### 消息队列

#### msgget

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

//成功返回 消息队列标识符，失败返回 -1
int msgget(key_t key, int msgflg);
```



#### msgsnd & msgrcv

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

//成功返回 0 ，失败返回 -1
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);

//成功返回 返回实际复制到 mtext 数组中的字节数，失败返回 -1
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp,
                      int msgflg);
```



#### msgctl

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

//成功返回 0 或其他值，失败返回 -1	
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
```







## chapter_14 多线程编程



### 线程和信号



#### `pthread_sigmask`

```c
#include <signal.h>

/*
how		表示要执行的操作
set		表示要操作的信号集。这个信号集中的信号将被添加到、移除或设置为线程的信号掩码。
oldset	用来保存之前的信号掩码状态
*/

//成功返回 0 ，失败返回错误码
int pthread_sigmask(int how, const sigset_t *set, sigset_t *oldset);
```



how:
- `SIG_BLOCK` ：将 set 中的信号添加到当前线程的信号掩码中，阻塞这些信号。
- `SIG_UNBLOCK` ：从当前线程的信号掩码中移除 set 中的信号，解除阻塞这些信号。
- `SIG_SETMASK` ：将当前线程的信号掩码设置为 set 中的信号。



#### `sigwait`

`sigwait` 函数会一直阻塞，直到 `set` 中的任何一个信号被发生。一旦其中一个信号被发生，`sig` 将被设置为对应的信号值

```c
#include <signal.h>

/*
set		表示要等待的信号集
sig		一个指向整数的指针，在信号发生时，将被设置为发生的信号值。
*/

//成功返回 0 ，失败返回 错误码
int sigwait(const sigset_t *set, int *sig);
```



#### 多线程中信号的使用

明确两点：

- 进程中的所有线程共享该进程的信号。线程库将根据线程掩码决定把信号发送给哪个具体的线程。这导致，假设某个线程触发信号，其他线程也会被收到。容易导致混乱。
- 所有线程共享信号处理函数。也就是说，当我们在某个线程中设置了某个信号的信号处理函数后，它将覆盖其他线程为同一个信号设置的信号处理函数。

基于这两点，我们想出应对策略：

- 不用信号处理函数来捕获信号。
- 我们将需要捕获的信号放进信号掩码，来屏蔽这些信号。
- 使用 `sigwait()` 函数来代替信号处理函数

这样做，

- 当某个线程触发信号时，由于信号被屏蔽，其他线程不会收到信号。
- 本线程可以利用`sigwait()` 函数来处理信号



详情见代码 `sigmask.c`











## chapter_15 进程池和线程池



在计算机组成原理中，一个程序在内存中占用的空间通常可以分为以下几个部分：

- **代码段**：也称为文本段或指令段，是存储程序执行指令的部分。代码段包含程序的机器指令，这些指令被处理器执行以完成程序的逻辑功能。代码段是只读的，因为程序在运行时不应该修改自身的指令。
- **数据段**：数据段存储程序的全局变量和静态变量。这些变量在程序的整个生命周期内保持不变，所以它们被存储在固定的内存位置。数据段可以分为初始化的数据（已经被预设初始值）和未初始化的数据（未设置初始值，通常会被初始化为零或者空值）。
- **堆**：堆是动态分配内存的区域，用于存储程序运行时动态分配的数据，如动态创建的对象和数据结构。堆的管理由程序员负责，通常需要手动分配和释放内存，否则可能导致内存泄漏或其他内存相关问题。
- **栈**：栈是用于存储函数调用的局部变量和函数调用信息的地方。每当一个函数被调用时，栈会分配一块内存用于存储该函数的局部变量和返回地址等信息。当函数调用结束后，栈上的这块内存会被释放，以便于下一个函数调用使用。
- **BSS 段：** 通常在数据段中有一个称为 BSS（Block Started by Symbol）的区域，用于存储未初始化的全局和静态变量。这些变量在程序启动时会被初始化为零或空值。
- **常量区（Constant Segment）：** 常量区存储程序中的常量数据，如字符串常量等。这些数据在程序运行期间保持不变。





调用 `fork()` 函数时会被复制的内容：

1. **进程内存空间：** 子进程会复制父进程的内存空间，包括代码段、数据段、堆和栈。但是，子进程和父进程各自的内存空间是独立的，修改一个进程的内存不会影响另一个进程的内存。
2. **文件描述符：** 子进程会继承父进程打开的文件描述符。这意味着子进程可以继续访问父进程打开的文件，但是文件偏移量是独立的，因此子进程和父进程可以在文件中独立移动读写位置。
3. **用户和组信息：** 子进程会继承父进程的用户ID和组ID。这在涉及权限和身份验证的操作中具有重要作用。
4. **进程的资源限制：** 子进程会继承父进程的资源限制，例如打开文件的最大数量、CPU时间限制等。
5. **信号处理程序：** 子进程会继承父进程设置的信号处理程序。
6. **进程控制信息：** 一些进程控制信息，例如进程的状态、进程号等，子进程也会继承。









## chapter_16 服务器调制、调试和测试















## chapter_17 系统监测工具

### lsof

lsof（list open file）是一个列出当前系统打开的文件描述符的工具。通过它我们可以了解进程打开了哪些文件描述符，或者文件描述符被哪些进程打开。

- `-i`  ：显示 socket 文件描述符，使用方法：
    - `lsof -i [46][protocol][@hostname|hostaddr][:service|port]`
    - `sudo lsof -i 4tcp@127.0.0.1:22` （注意不能有空格）
- `-u` ：显示指定用户启动的所有进程打开的所有文件描述符
- `-c` ：显示指定的命令打开的文件描述符
- `-p` ：显示指定进程打开的所有文件描述符
- `-t` ：显示打开了目标文件描述符的进程的 PID



### nc

nc（netcat）主要被用来快速构建网络连接。它可以用于在网络上进行数据传输、监听端口、创建基本的网络服务等。`nc` 可以作为一个工具来测试网络连接、传输文件，或者作为一个简单的服务器进行网络通信。

- `-l`：监听模式，用于创建服务器端。
- `-p <port>`：指定要监听或连接的端口号。
- `-u`：使用 UDP 协议进行通信。
- `-e <program>`：在服务器模式下执行指定的程序来处理连接。
- `-q <seconds>`：设置服务器模式下的超时时间，单位为秒。
- `-v`：显示详细输出，可用多次以增加详细级别。
- `-n`：禁用主机名解析。
- `-i <seconds>`：设置数据传输的间隔时间，单位为秒。
- `-w <seconds>`：设置超时时间，用于连接或发送等待。
- `-z`：仅进行扫描，不进行实际连接。
- `-s <source_ip>`：指定源 IP 地址。
- `-d`：开启调试模式。
- `-h` 或 `--help`：显示帮助信息。
- `-v` 或 `--version`：显示版本信息。



### awk

`awk` 是一种强大的文本处理工具，通常用于从文件或标准输入流中提取数据、转换数据格式和执行其他文本处理任务。它以行为单位逐行扫描文本文件，并按照用户指定的规则进行处理。

- `awk '{ print $1 }' file.txt` ：提取文件中每行的第一个字段，默认使用空格作为字段分隔符
- `-F`：指定字段分隔符



### cut

`cut` 命令通常用于从文本文件或标准输入中提取特定字段。以下是一些常用的 `cut` 命令选项：

1. `-c, --characters=LIST`：按字符位置切割。列表中的字符位置可以是单个数字，用逗号分隔的数字范围（例如：1,3-5）或者以逗号分隔的多个字符位置。
2. `-f, --fields=LIST`：按字段切割。列表中的字段可以是单个数字，用逗号分隔的数字范围（例如：1,3-5）或者以逗号分隔的多个字段。
3. `-d, --delimiter=DELIM`：指定字段分隔符。默认情况下，`cut` 使用制表符作为字段分隔符。
4. `-s, --only-delimited`：仅显示包含字段分隔符的行。默认情况下，`cut` 会显示不包含字段分隔符的行。
5. `--complement`：显示除了指定字段之外的字段。默认情况下，`cut` 显示指定字段。
6. `-n`：关闭分隔符字符解释。默认情况下，`cut` 将分隔符视为正则表达式。

`sudo netstat -antp | grep 12345 | awk '{print $7}' | cut -d '/' -f 1 | xargs sudo kill`



### 实际问题

- 查看特定端口号被哪个进程打开：
    - `lsof -i:<port>` ：`sudo lsof -i:22`
- 查看特定进程打开哪些端口号：（好像都不太行）
    - `lsof -i -a -p <PID>`
    - `netstat -antp | grep `
- 杀死特定进程：
    - `ps -ef | grep 12345 | awk '{print $2}' | xargs -I {}  kill {}`

