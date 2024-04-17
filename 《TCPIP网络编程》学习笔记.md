<h1>《TCP/IP网络编程》学习笔记</h1>

## Chapter_01 理解网络编程和套接字

### Socket函数原型

```c
#include <sys/socket.h>

//成功返回文件描述符，失败返回 -1
int socket(int domain, int type, int protocol);

//成功返回 0， 失败返回 -1
int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);

//成功返回 0，失败返回 -1
//backlog 挂起的连接队列的最大长度
int listen(int sockfd, int backlog);

//成功返回文件描述符，失败返回 -1
//如果在没有连接请求的情况下调用该函数，则不会返回，直到有连接请求为止
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);

//成功时返回 0，失败时返回 -1
int connect(int sockfd, struct sockaddr *serv_addr, socklen_t addrlen);
```

### 底层文件I/O(系统I/O)函数原型

	学习和使用这些API最快的途径是利用系统自带的man查看手册，查看系统IO可以用`man 2 open`， 查看标准I/O可以用`man 3 fopen`。

>关于linux中man 1 2 3 … 的区别 :
>1、Standard commands （标准命令）
2、System calls （系统调用）
3、Library functions （库函数）
4、Special devices （设备说明）
5、File formats （文件格式）
6、Games and toys （游戏和娱乐）
7、Miscellaneous （杂项）
8、Administrative Commands （管理员命令）
9、 其他（Linux特定的）， 用来存放内核例行程序的文档。

常用的系统I/O接口有：
```c
open()
read()
write()
close()
lseek()
```

#### `open()`
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

//调用成功时返回一个文件描述符fd,失败时返回-1，并设置errno

int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);

int creat(const char *pathname, mode_t mode);

int openat(int dirfd, const char *pathname, int flags);
int openat(int dirfd, const char *pathname, int flags, mode_t mode);
```

#### `read()`

```c
#include <unistd.h>

/*
我好像懂了，read 应该是只有在读到 EOF 时才会返回 0
才会结束 while 循环。而没有数据读时会堵塞，不会返回 0
所以当服务器和客户端读数据时都用wihle循环的话，没人发送 EOF 信号，就都堵塞住了
*/

//文件描述符 存储数据的缓冲地址 要读取数据的字节数
//成功时返回读取的字节数,出错时返回EOF，读到文件末返回0
ssize_t read(int fd, void *buf, size_t count);
```

#### `write()`

```c
#include <unistd.h>

//文件描述符 要写入数据的缓冲地址 要写入数据的字节数，不大于buf
//成功时返回写入的字节数,出错时返回EOF，读到文件末返回0
ssize_t write(int fd, const void *buf, size_t count);
```

#### `close()`

```c
#include <unistd.h>

//关闭成功时返回0，出错时返回EOF
int close(int fd);
```

#### `lseek()`

```c
#include <sys/types.h>
#include <unistd.h>

//文件描述符 偏移量，可正可负 指定一个基准点，基准点+偏移量等于当前位置
//成功时则返回目前的读写位置, 也就是距离文件开头多少个字节
//出错时返回-1, errno 会存放错误代码.
off_t lseek(int fd, off_t offset, int whence);
```

### 标准I/O函数原型

	与系统I/O不同的是，标准I/O带有缓冲区，可以减少系统调用，提高系统效率。

使用标准I/O只需包含头文件：`#inlcude <stdio.h>`
常用的标准I/O接口有：

```c
fopen()
fread()
fwrite()
fclose()
```

#### `fopen()`

			  ┌─────────────┬───────────────────────────────┐
	          │fopen() mode │ open() flags                  │
	          ├─────────────┼───────────────────────────────┤
	          │     r       │ O_RDONLY                      │
	          ├─────────────┼───────────────────────────────┤
	          │     w       │ O_WRONLY | O_CREAT | O_TRUNC  │
	          ├─────────────┼───────────────────────────────┤
	          │     a       │ O_WRONLY | O_CREAT | O_APPEND │
	          ├─────────────┼───────────────────────────────┤
	          │     r+      │ O_RDWR                        │
	          ├─────────────┼───────────────────────────────┤
	          │     w+      │ O_RDWR | O_CREAT | O_TRUNC    │
	          ├─────────────┼───────────────────────────────┤
	          │     a+      │ O_RDWR | O_CREAT | O_APPEND   │
	          └─────────────┴───────────────────────────────┘

```c
//成功时返回一个文件流指针：FILE* 失败时返回NULL
FILE *fopen(const char *pathname, const char *mode);

FILE *fdopen(int fd, const char *mode);

//freopen() 的主要用途是更改与标准文本流（stderr、stdin或stdout）关联的文件。
FILE *freopen(const char *pathname, const char *mode, FILE *stream);
```

#### `fread()`

```c
//接收数据的地址，最小尺寸size*nmemb字节 每次读取的一个单元的大小，单位字节
//读取的最大单元个数 即将读取文件的文件流
//成功时返回实际读取的单元个数 文件结束或读取出错时返回0或一个小于nmemb的数
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
```

#### `fwrite()`

```c
//要写入文件的数据的地址 每次写入的一个单元的大小
//写入的单元个数 即将写入文件的文件流
//成功时返回实际写入的单元个数 出错时返回一个与nmemb不同的数
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
```

#### `fclose()`

```c
//stream: 要关闭的文件流指针
//成功时返回0 失败时返回EOF，并设置errno
int fclose(FILE *stream);
```

### 系统IO与标准IO的区别

最大的区别在于：
- 标准IO引用了缓冲机制，在每次执行标准IO操作时，所有的操作都在缓存区中执行，在系统空闲时或者缓冲区满的时候再写入物理磁盘，从而提高效率。
- 系统IO没有缓冲机制，每次执行系统IO操作时都会进行硬盘的读写

### 关于`ssize_t`、`size_t`等数据类型

这些都是元数据类型，在 `sys/types.h` 头文件中一般由 `typedef` 定义。因此只是起了别名。
因为操作系统的不同，数据类型的表现形式也随之改变，比如`int`可能时32位，也可能是16位。当需要修改程序中使用的数据类型时，我们只需要修改并编译声明头文件即可。为了与程序员定义的新数据类型加以区分，操作系统定义的数据类型会添加后缀`_t`。

### P24 习题

1. 套接字在网络编程中的作用是什么？为何称它为套接字？

	网络编程是编写程序使两台连网的计算机相互交换数据。套接字是网络数据传输用的软件设备。
	编程中的“套接字”是用来连接网络的工具。它本身就带有“连接”的含义。

2. 在服务器端创建套接字后，会依次调用listen函数和accept函数。请比较说明二者作用。

	>socket() //创建套接字（安装电话机）
	>bind() //分配IP地址和端口号（分配电话号码）
	>listen() //转为可接受请求状态（连接电话线）
	>accept() //接受理客户端的连接请求（拿起话筒进行对话
	
3. Linux中，对套接字数据进行`I/O`时可以直接使用文件`I/O`相关函数；而在Windows中则不可以。原因为何？

	对于Linux，一切皆文件。
	Windows中，区分文件句柄和套接字句柄。

4. 创建套接字后一般会给它分配地址，为什么？为了完成地址分配需要调用那个函数？

	要在网络上区分来自不同机器的套接字，所以需要地址信息。
	分配地址是通过bind()函数实现
	
5. Linux中的文件描述符与Windows的句柄实际上非常类似。请以套接字为对象说明他们的含义。

	Linux的文件描述符是为了区分指定文件而赋予文件的整数值（相当于编号）。Windows的文件描述符其实也是套接字的整数值，其目的也是区分指定套接字。

6. 底层文件I/O函数与ANSI标准定义的文件I/O函数之间有何区别？

	ANSI标准定义的输入、输出函数是与操作系统（内核）无关的以C标准写成的函数。相反，底层文件I/O函数是直接提供的。理论上ANSI标准I/O提供了某些机制，性能上优于底层I/O

7. 参考本书给出的示例low_open.c和low_read.c,分别利用底层文件I/O和ANSI标准I/O编写文件复制程序。可任意指定复制程序的使用方法

	见  filecopy_sys.c  和  filecopy_std.c



## Chapter_02 套接字类型与协议设置

### Socket函数详解

```c
#include<sys/socket.h>

//成功时返回文件描述符，失败时返回 -1
int socket(int domain, int type, int protocol);

/*
domain: 套接字中使用的协议族(Protocol Family)信息
type: 套接字数据传输类型信息
protocol: 计算机间通信中使用的协议信息
*/
```

### 协议族(Protocol Family)

<table border="1">
	<caption>头文件<code>sys/spcket.h</code>中声明的协议族</caption>
    <tr>
        <th>名称</th> <th>协议族</th>
    </tr>
    <tr>
        <td>PF_INET</td> <td>IPv4互联网协议族</td>
    </tr>
    <tr>
        <td>PF_INET6</td> <td>IPv4互联网协议族</td>
    </tr>
    <tr>
        <td>PF_LOCAL</td> <td>本地通信的UNIX协议族</td>
    </tr>
    <tr>
        <td>PF_PACKET</td> <td>底层套接字的协议族</td>
    </tr>
    <tr>
        <td>...</td> <td>...</td>
    </tr>
</table>


### 套接字类型(Type)

套接字类型指的是套接字的数据传输方式。一个协议族可能存在多种数据传输方式

#### 套接字类型1：面向连接的套接字(SOCK_STREAM)

其实就是 IPPROTO_TCP，特征：

- 传输过程中数据不会消失
- 按序传输数据
- 传输的数据不存在数据边界

收发数据的套接字内部有缓冲，简言之就是字节数组。通过套接字传输的数据将保存到该数组。因此，收到数据并不意味着马上调用read函数。**read函数和write函数的调用次数并无太大意义**。所以说面向连接的套接字去不存在数据边界。

用一句话概括面向连接的套接字：
>可靠的、按序传递的、基于字节的面向连接的数据传输方式的套接字

#### 套接字类型2：面向消息的套接字(SOCK_DGRAM)

其实就是 IPPROTO_UDP，特征：

- 强调快速传输而非传输顺序
- 传输的数据可能丢失也可能损毁
- 传输的数据有数据边界
- 限制每次传输的数据大小

**存在数据边界意味着接收数据的次数应和传输次数相同**。

用一句话概括面向消息的套接字：
>不可靠的、不按序传递的、以数据的高速传输为目的的套接字



### 协议的最终选择

因为可能存在：同一协议族中存在多个数据传输方式相同的协议
数据传输方式相同，但协议不同。此时需要通过第三个参数具体指定协议信息。

- 创建套接字：IPv4协议族中面向连接的套接字
	
	`int tcp_socket = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);`

- 创建套接字：IPv4协议族中面向消息的套接字
	
	`int udp_socket = socket(PF_INET, SOCK_DGRAM, IPPROTO_UDP);`



### P35 习题

1. 什么是协议？在收发数据中定义协议有何意义？

	协议就是为了完成数据交换而定好的约定。
	因此，定义协议意味着对数据传输所必需的的承诺进行定义。

2. 面向连接的TCP套接字传输特性有3点，请分别说明。

	- 传输过程中数据不会消失
	- 按序传输数据
	- 传输的数据不存在数据边界

4. 下列数据适合用哪类套接字传输？并给出原因。

	a. 演唱会现场直播的多媒体数据（UDP）
	b. 某人压缩过的文本文件（TCP）
	c. 网上银行用户与银行之间的数据传递（TCP）
	
5. 何种类型的套接字不存在数据边界？这类套接字接收数据时需要注意什么？

	连接指向型TCP套接字不存在数据边界。因此输入输出函数的响应次数不具有意义。重要的不是函数的响应次数，而是数据的收发量。因此，必须将传输数据的量和接收数据的量制作成编码，保证发送数据的量和接收数据的量是一致的，特别要注意是制作依赖函数响应次数判断代码
	
	

## Chapter_03 地址族与数据序列

这章学的我头大，nmmd，弃了弃了

### 表示IPv4地址的结构体

```c
/*
成员分析：
sin_family 地址族
sin_port 以网络字节序保存，大端序(big endian)
sin_addr 以网络字节序保存, unit32_t,当作32位整数即可
sin_zero无特殊含义，需填充为0，这也是 memset(&serv_addr, 0, sizeof serv_addr)的目的

*/

struct sockaddr_in
{
    sa_family_t sin_family; //地址族
    unit16_t sin_port; //16位TCP/UDP端口号
    struct in_addr sin_addr; //32位IP地址
    char sin_zero[8]; //不使用
};

struct in_addr
{
    in_addr_t s_addr; //32位IPv4地址 uint32_t
};
```

```c
struct sockaddr
{
    sa_family_t sin_family;
    char sa_data[14]; //地址信息
    //只有14个字节怎么放得下的啊，输出了试了下，好像在骗人
}

struct sockaddr_in serv_addr;

//注意要转换
bind(serv_sock, (struct sockaddr *) &serv_addr, sizeof serv_addr)
```

### 网络字节序与地址变换

远离这玩意，否则会变得不幸

- 大端序（Big Endian）：高位字节存放到低位地址
- 小端序（Little Endian）：高位字节存放到高位地址

主机字节序（Host Byte Order）一般为 小端序
网络字节序（Network Byte Order）统一为大端序

#### 字节序转换（Endian Conversions）

```c
/*
h host
n network
s short 2字节
l long 4字节(linux下)
*/

unsigned short htons(unsigned short);//把short型数据从主机字节序转化为网络字节序
unsigned short ntohs(unsigned short);
unsigned long htonl(unsigned long);
unsigned long ntohl(unsigned long);
```


### 网络地址的初始化与分配

#### 将字符串信息转化为网络字节序的整数型

```c
#include <arpa/inet.h>

//成功返回32位大端序整数型值，失败时返回 INADDR_NONE
in_addr_t inet_addr(const char *string);

//成功返回 1，失败返回 0
int inet_aton(const char *string, struct in_addr *addr);
```

#### 将网络字节序的整数型转化为字符串信息

```c
#include <arpa/inet.h>

//成功返回转换的字符串地址值，失败时返回 -1
//可以看到返回的是指针，但是函数并没有向我们申请空间
//再次调用 inet_ntoa 函数，内存空间中的结果会被覆盖
//因此在获得结果后，我们要记得将结果再放到自己分配的空间中
char *inet_ntoa(struct in_addr adr);
```

### P57 习题

1. IP地址族IPv4和IPv6有何区别？在何种背景下诞生了IPv6?

	IPV4是4字节地址族，IPV6是16字节地址族。
	IPV6的诞生是为了应对2010年前后IP地址耗尽的问题而提出的标准。

2. 通过IPV4网络ID、主机ID及路由器的关系说明向公司局域网中的计算机传输数据的过程。

	首先数据传输的第一个环节是向目标IP所属的网络传输数据。此时使用的是IP地址中的网络ID。传输的数据将被传到管理网络的路由器，接受数据的路由器将参照IP地址的主机号找自己保存的路由表，找到对应的主机发送数据
	
3. 套接字地址分为IP地址和端口号。为什么需要IP地址和端口号？或者说，通过IP可以区分哪些对象？通过端口号可以区分哪些对象？

	IP地址是为了区分网络上的主机。端口号是区分同一主机下的不同的SOCKET，以确保软件准确收发数据。


4. 说出下面这些IP地址的分类。

- 214.121.212.102	(C)
- 120.101.122.89	(A)
- 129.78.102.211	(B)

5. 计算机通过路由器或交换机连接到互联网。请说出路由器和交换机的作用

	路由器是帮助数据传输到目的地的中介。不仅如此，还起到帮助连接本地网络的电脑和互联网的作用
	- 控制层：生成路由转发表
	- 数据层：根据转发表转发数据

6. 什么是知名端口？其范围是多少？知名端口中具有代表性的HTTP合同FTP端口号各是多少？

	“知名端口(Well-known PROT)”是指预定分配给特定操作的端口。其范围是0~1023。
	HTTP：80端口 和 FTP：20 或21
	
7. bind()函数调用时，第二个传入的参数类型与函数原型不同，请说明原因。

	bind函数第二个参数类型是sockaddr结构体，很难份分配IP地址和端口号，因此IP地址和PORT号的分配是通过sockaddr_in完成的。因为该结构体和sockaddr结构体的组成字节序和大小完全相同，所以可以强转

8. 请解释大端序、小端序、网络字节序，并说明为何需要网络字节序
	
	小端序是把高位字节存储到高位地址上；大端序是把高位字节存储到低位地址上。因为保存栈的方式有差异，所以对网络传输数据的过程制定了标准，这就是“网路字节序”。而且，在网络字节序中，数据传输的标准是“大端序”

9. 大端计算机希望将4字节整型数据12传到小端序计算机。请说出数据传输过程中发生的字节序变换过程

	因为网络字节序的顺序标准是“大端序”，所以大端序的计算机在网络传输中不需要先转换字节顺序，直接传输。但是接受数据的是小端序计算机，因此，要经过网络转本地序的过程，再保存到存储设备上
	
10. 怎么表示回送地址？其含义是什么？如果向回送地址传输数据将会发生什么情况？

	回送地址表示计算机本身，为127.0.0.1。因此，如果将数据传送到IP地址127.0.0.1，数据会不会通过传输到网络的其他设备上而直接返回。



## Chapter_04 基于TCP的服务器端/客户端（1）

滑铁卢了，遭遇未知阻塞 :neutral_face:

### 进入等待连接请求状态

```c
#include <sys/socket.h>

/*
只有调用了 listen 函数，客户端才能进入可发出连接请求的状态
也就是说，才能调用 connect 函数，若提前调用则会发生错误

sock 希望进入等待连接请求状态的套接字文件描述符，
	传入的描述符会成为服务器端套接字（监听套接字）
backlog 连接请求等待队列的长度
*/

//成功返回 0，失败返回 -1
int listen(int sock, int backlog);
```

### 受理客户端连接请求

```c
#include <sys/socket.h>

/*
accept 会执行两个动作
	1. 主服务器进程为每一个新的连接请求创建一个新的套接字（连接套接字），并将标识符返回给发起连接的客户
	2. 主服务器进程还要创建一个从属服务器进程来处理新建立的连接
这样，从属服务器进程用新创建的套接字和客户进程建立连接，主服务器进程继续用原来的套接字重新调用accept，继续接受下一个连接请求。

值得注意的是，传入的第二个参数是空结构体（未初始化），函数会将填充客户端地址信息进去
而 conncet 函数中，传入的第二个参数已经赋值，函数需要这个信息
*/

//成功时返回创建的套接字文件描述符，失败时返回 -1
int accept(int sock, struct sockaddr *addr, socklen_t *addrlen);
```

### 发起连接请求

```c
#include <sys/socket.h>

/*
sock 客户端套接字文件描述符
servaddr 保存目标服务器端地址信息的变量地址值
addrlen 第二个参数的变量长度

通过观察书中的 echo_client.c 程序， connect() 的位置放在循环外，
所以这个函数是个异步函数之类的？？？
破案了，需要重新运行客户端
*/

//成功时返回 0，失败时返回 -1
int connect(int sock, struct sockaddr *servaddr, socklen_t addrlen);
```

### P81 习题

1. 请说明TCP/IP的4层协议栈，并说明TCP和UDP套接字经过的层级结构差异

	链路层—>IP层—>TCP层—>应用层
	链路层—>IP层—>UDP层—>应用层
	
2. 请说出TCP/IP协议栈中链路层和IP层的作用，并给出两者关系。

	链路层是LAN、WAN、MAN等网络标准相关的协议栈，是定义物理性质标准的层级。相反，IP层是定义网络传输数据标准的层级。即IP层负责以链路层为基础的数据传输
	
3. 为何需要把TCP/IP协议栈分成4层（或7层）？结合开放式系统回答

	将复杂的TCP/IP协议分层化的话，就可以将分层的层级标准发展成开放系统。实际上，TCP/IP是开放系统，各层级都被初始化，并以该标准为依据组成了互联网。因此，按照不同层级标准，硬件和软件可以相互替代，这种标准化是TCP/IP蓬勃发张的依据
	
4. 客户端调用connect函数向服务器端发送连接请求。服务器端调用哪个函数后，客户端可以调用connect函数？

	listen函数，客户端才可以调用connect函数

5. 什么时候创建连接请求等待队列？它有何作用？与accept有什么关系

	listen函数的调用创建了请求等待队列。它是存储客户端连接请求信息的空间。accept函数调用后，将从本地存储的连接请求信息取出，与客户端建立连接。
	
6. 客户端中为何不需要调用bind函数分配地址？如果不调用bind函数，那何时、如何向套接字分配IP地址和端口号？

	客户端的IP地址和端口在调用 connect 函数时自动分配，无需调用标记的 bind 函数进行分配。
	调用 connect 函数时，在内核中，IP用计算机的IP，端口随机。

## Chapter_05 基于TCP的服务器端/客户端（2）

肯定是代码的问题而不是我的问题，都照抄了还是没有预期结果

没啥好写的，TCP连接的三个阶段：**连接建立、数据传送和连接释放**
看《计算机网络就行了》

不过这章的计算器代码挺不错的，数据 char 到 int 的转换，感觉考验功底。
不过我已经不想写了。

### P99 习题

1. 请说明TCP套接字连接设置的三次握手过程。尤其是3次数据交换过程每次收发的数据内容。

	见 我草稿本 :neutral_face:

2. TCP是可靠的数据传输协议，但在通过网络通信的过程可能丢失数据。请通过ACK和SEQ说明TCP通过何种机制保证丢失数据的可靠传输。

	停止等待协议和ARQ协议
	
3. TCP套接字中调用write和read函数时数据如何移动？结合I/O缓冲进行说明

	当write函数被调用时，数据就会向端口的输出缓冲区移动。然后经过网络传输传输到对方主机套接字的输入缓冲。这样，输入缓冲中存储的数据通过read函数的响应来读取
	
4. 对方主机的输入缓冲剩余50字节空间时，若本方主机通过write函数请求传输70字节，问TCP如何处理这种情况？

	对方主机会把输入缓冲中可存储的数据大小传送给要传输数据的数据（本方）。因此，在剩余空间为50字节的情况，即使要求传送70字节的数据，也不能传输50字节以上，剩余的部分保存在传输方的输出缓冲中，等待对方主机的输入缓冲出现空间。而且，这种交换缓冲多余空间信息的协议被称为滑动窗口协议
	
5. 芜湖，我写完了

6. 芜湖，我太厉害了

## chapter_06 基于UDP的服务器端/客户端

1. UDP并非每次都快于TCP，TCP比UDP慢的原因通常有两点：

	- 收发数据前后进行的连接设置及清除过程
	- 收发数据过程中为保证可靠性而添加的流控制

	如果收发的数据量小但需要频繁连接，UDP比TCP更高效
	
2. UDP服务器和客户端均只需一个套接字，一个套接字可以和多台主机通信

### 基于UDP的数据I/O函数

创建好TCP套接字后，传输数据时无需再添加地址信息，因为TCP套接字将保持与对方套接字的连接。换言之，TCP套接字知道目标地址信息。

但UDP套接字不会保持连接状态，因此每次传输数据都要添加目标地址信息。

#### sendto 函数

```c
#include <sys/socket.h>

/*
sock 套接字文件描述符
buff 保存待传输数据的缓冲地址值
nbytes 待传输的数据长度，以字节为单位
flags 可选项参数， 没有则传递 0
to 存有目标地址信息的 sockaddr 结构体变量的地址值
addrlen 传递给参数 to 的地址值结构体变量长度（倒数第二个参数的size）
*/

//成功时返回传输的字节数，失败时返回 -1
ssize_t sendto(int sock, void *buff, size_t nbytes, int flags,
              struct sockaddr *to, socklen_t addrlen);
```

1. 调用 sendto 函数传输数据前应完成对套接字的地址分配工作（分配IP和Port），如果尚未分配地址信息，则在首次调用 sendto 函数时给相应套接字自动分配IP和端口。（我怎么觉得每次调用都会分配地址呢，毕竟三阶段呢）
	sendto 函数三阶段：
	- 第一阶段：向UDP套接字注册目标IP和端口号
	- 第二阶段：传输数据
	- 第三阶段：删除UDP套接字中注册的目标地址信息

2. 综述，调用 sendto 函数时自动分配 IP 和端口号
3. 调用 sendto 函数前，sendto 函数的目标主机程序需已经开始运行

#### recvfrom 函数

```c
#include <sys/socket.h>

/*
sock 套接字文件描述符
buff 保存接收数据的缓冲地址值
nbytes 可接受的最大字节数，故无法超过参数buff所指的缓冲大小
flags 可选项参数， 没有则传递 0
to 存有发送端地址信息的 sockaddr 结构体变量的地址值
addrlen 保存参数 from 的结构体变量长度的变量地址值（倒数第二个参数的size 的地址值）

调用 recvfrom 函数可获取数据传输端的地址
*/

//成功时返回接受的字节数，失败时返回 -1
ssize_t recvfrom(int sock, void *buff, size_t nbytes, int flags,
                struct sockaddr *from, socklen_t *addrlen);
```


#### 创建已连接UDP套接字

1. UDP套接字默认属于未连接套接字
2. 创建已连接UDP套接字只需调用 connect() 函数（TCP：你这不是NTR嘛你）
3. 调用 connect() 函数并不意味着要与对方套接字连接，这只是向UDP套接字注册目标IP和端口信息
4. 之后，sendto 函数只会传输数据，也可以用 write 和 read 代替 sendto 和 recvfrom（TCP:你还说你不是NTR :sob:）

### P117 习题

1. UDP为什么比TCP速度快？为什么TCP数据传输可靠而UDP数据传输不可靠？

	UDP和TCP不同，不进行流量控制。由于该控制涉及到套接字的连接和结束，以及整个数据收发过程，因此，TCP传输的数据是可以信赖的。相反,UDP不进行这种控制，因此无法信任数据的传输，但正因UDP不进行流量控制，所以比TCP更快
	
2. UDP数据包向对方主机的UDP套接字传递过程中，IP和UDP分别负责哪些部分？

	IP负责链路选择。UDP负责端到端的传输

3. UDP一般比TCP快，但根据交换数据的特点，其差异可大可小。请说明何种情况下UDP的性能优于TCP
	
	UDP与TCP不同，不经过连接以及断开SOCKET的过程，因此，在频繁的连接及断开的情况下，UDP的数据收发能力会凸显出更好的性能。(单次传输数据量小，且需频繁传输)

4. 客户端TCP套接字调用connect函数时自动分配IP和端口号。UDP中不调用bind函数，那何时分配IP和端口号？

	首次调用sendto函数时，发现尚未分配信息，则给相应的套接字自动分配IP和端口号

5. TCP客户端必须调用connect函数，而UDP中可以选择性调用。请问，在UDP中调用connect函数有哪些好处？

	每当以UDP套接字为对像调用sendto函数时，都要经过以下过程
	
	- 第一阶段：为目标UDP注册端口和IP
	- 第二阶段：数据传输
	- 第三阶段：删除UDP注册的IP和端口信息
	
	其中，只要调用connect函数，就可以忽略每次传输数据时反复进行的第一阶段和第三阶段。然而，调用connect函数并不意味着经过连接过程，只是将IP地址和端口号指定在UDP的发送对象上。这样connect函数使用后，还可以用write、read函数进行数据处理，而不必使用sendto、recvfrom
	
6. 见 udp_server.c 和 udp_client.c



## chapter_07 优雅地断开套接字连接

TCP中的断开连接过程比建立连接过程更重要，因为连接过程中一般不会出现大的变数，但断开过程中可能发生预想不到的情况。

1. 一旦两台主机建立套接字连接，每个主机就会拥有单独的输入流和输出流，一个主机的输入流和另一台主机的输出流相连。
2. 调用 close 函数会同时断开输入输出流，导致无法调用与接收数据相关的函数。

### shutdown 函数

```c
#include <sys/socket.h>

/*
sock 需要断开的套接字文件描述符
howto 传递断开方式信息
	SHUT_RD 断开输入流
	SHUT_WR 断开输出流
	SHUT_RDWR 同时断开I/O流
*/

//成功返回 0，失败时返回 -1
int shutdown(int sock, int howto);
```

- 若传递 SHUT_RD，则断开输入流，套接字无法接收数据。即使输入缓冲收到数据也会抹去。
- 若传递 SHUT_WR，则中断输出流，也就无法传输数据。但如果输出缓冲还留有未传输的数据，则将传递至目标主机。
- 若传入 SHUT_RDWR，则同时中断I/O流。相当于分两次调用 shut_down 函数

很好理解啦，TCP缓存是在运输层，socket 是软件，也就是在应用层。socket 关闭则应用层关闭，输入缓冲也就无法读进socket。但留存在输出缓冲的数据，发送是由运输层决定。

### P127 习题

1. 解释TCP中“流”的概念。UDP中能否形成流？请说明原因

	TCP的流指，两台主机通过套接字建立连接后进入可交换数据的状态，也称为“流形成的状态”。而对于UDP来说，不存在流，因为两个SOCKET不能相互连接
	
2. Linux中的close函数或Windows中的closesocket函数属于单方面断开连接的方法，有可能带来一些问题。什么是单方面断开连接？什么情况下会出现问题?

	单方面的断开连接意味着套接字无法再收发数据。
	一般在对方有剩余数据未发送完成时，断开己方连接，会造成问题。

3. 什么是半关闭？针对输出流执行半关闭的主机处于何种状态？半关闭会导致对方主机接收什么信息？

	半关闭是指只断开输入和输出流中的一个。
	如果对输出流进行半关闭，己方套接字无法传送数据，但可以接收对方主机传送的数据。
	对方主机会接收到 EOF 信息。
	

## chapter_08 域名及网络地址

### 知识点引入

在编写客户端需要使用服务器提供的服务时，当然不能要求客户去输入服务器的IP和端口，由于提供服务的IP可能会频繁变动，相比较，域名会稳定不变，因此利用域名编写函数会比利用IP编写好一些。每次运行程序时，根据域名获取IP地址，再接入服务器，这样程序就不会依赖于服务器IP地址了。

### 利用域名获取IP地址

```c
#include <netdb.h>

//成功时返回 hostent 结构体地址， 失败时返回 NULL 指针
struct hostent * gethostbyname(const char *hostname);
```

```c
struct hostent
{
    char *h_name; 		//official name
    char **h_aliases; 	//alias list
    int h_addrtype;		//host address type
    int h_length; 		//address length
    char **h_addr_list;	//address list
}
```

IP 和域名是多对多关系

    h_name 官方域名
    h_aliases 除官方域名外的其他域名
    h_addrtype 函数支持IPv4 和 IPv6，若是IPv4，变量会存有 AF_INET
    h_length IP地址长度，IPv4是4个字节，IPv6是16个字节（128位）
    h_addr_list 保存域名对应IP地址

### 利用IP地址获取域名

```c
#include <netdb.h>

/*
addr 含有IP地址信息的 in_addr 结构体指针（addr.sin_addr）。
	为了同时传递IPv4地址之外的信息，该变量的类型声明为char指针(我看到是 void *)
len 第一个参数的字节数，IPv4时为 4，IPv6时为 16
family 地址族信息，AF_INET AF_INET6
*/

//成功时返回 hostent 结构体地址， 失败时返回 NULL 指针
struct hostent * gethostbyaddr(const char *addr, socklen_t len, int family);
```

### P138 习题

1. 在浏览器地址栏输入 www.bai.com ，并整理出主页显示过程。假设浏览器访问的默认DNS服务器中并没有关于 www.baidu.com 的IP地址信息

	1. 计算机向DNS服务器询问IP地址
	2. 默认DNS服务器没有IP地址信息，因此向DNS主机发出询问
	3. DNS查询服务器从其更上级的DNS服务器接收IP地址信息
	4. DNS查询服务器将查到的IP地址逐级返还给主机
	5. 网络浏览器访问接收到的IP地址网站


## chapter_09 套接字的多种可选项

表见 P140

### getsockopt & setsockopt

```c
#include <sys/socket.h>

/*
level 协议层，如 SOL_SOCKET,IPPROTO_IP,IPPROTO_TCP
optname 可选项名，如 SO_SNDBUF,SO_REVBUF,SO_REUSEADDR
optval 保存查看结果的缓冲地址值（好像都是int型）
optlen 可选项信息的字节数
*/

//成功返回 0，失败返回 -1
int getsockopt(int sock, int level, int optname, void *optval,
               socklen_t *optlen);
```

```c
#include <sys/socket.h>

/*
level 协议层，如 SOL_SOCKET,IPPROTO_IP,IPPROTO_TCP
optname 可选项名，如 SO_SNDBUF,SO_REVBUF,SO_REUSEADDR
optval 保存查看结果的缓冲地址值（好像都是int型）
optlen 可选项信息的字节数

注意最后一个参数的区别哦
*/

//成功返回 0，失败返回 -1
int setsockopt(int sock, int level, int optname, void *optval,
               socklen_t optlen);
```

### SO_SNDBUF & SO_RCVBUF

SO_SNDBUF & SO_RCVBUF 是输出/输入缓冲大小的可选项

### SO_REUSEADDR

如其名，套接字可重新使用

在断开连接阶段，主动关闭的一方（A）会有 TIME_WAIT 阶段，时间为 2MSL(Maximum Segment Lifetime - 最大报文段寿命)。

理由有二：
- 为了保证 A 发送的最后一个 ACK 报文段能够到达 B（被动关闭的一方）。B 只有接收到 ACK 报文段才会进入 CLOSED 阶段，如果 B 没收到 ACK 报文，B 会重传 FIN 报文。如果 A 不等待这一段时间，而是在发送完 ACK 报文段后立即释放连接，那么 B 有可能无法收到 ACK 报文段而无法进入 CLOSED 阶段
- 防止“已失效的连接请求报文段”出现在本连接中。经过 2MSL，可以使本连接持续的时间内所产生的所有报文段都从网络中消失。

由此可知，如果 server 端主动断开连接，再使用同一端口重新运行时，将输出 "bind() error" 消息。
但为啥 client 端主动断开连接之后可以重新运行呢？因为客户端套接字的端口号是任意指定的，客户端每次运行程序都会动态分配端口号。



那么我们对于套接字分配地址信息应该有所了解了：

- 首先，服务端的主服务器用的套接字（监听套接字）地地址信息是由 bind() 函数来分配的，用我们指定的 IP 和端口号分配。
- 其次，每建立一个连接，accept() 函数都会建立新的套接字（连接套接字），并且动态地分配地址信息。
- 对于客户端，在调用 connect() 函数时，由内核动态分配套接字。



另外明确一点，在服务器程序中的 clnt_addr 的地址信息，就是我们所说的 连接套接字，是由 accept() 分配地址信息的，占用的是服务器的端口号，并不是客户端程序的那边的地址信息。

同样的，客户端程序中的 serv_addr 的地址信息由 connect() 分配，占用的是客户端的端口号。

那么连接从哪体现呢？也就是说，服务器中的 clnt_sock 是如何与 客户端的 sock 建立连接的呢？

	回看 UDP 的服务器端/客户端，因为没有建立连接，所以每次调用 sendto()/recvfrom() 函数时都需要将另一方的地址信息传参进去。但其实，也只不过是在调用 sendto() 函数时，由内核（应该是吧）动态分配了地址信息。
	双方的套接字是如何知道对方的呢，数据的目的地如何知晓的呢？要传送数据到对方主机应用进程，那么必须知道 IP+端口号，所以问题就在，对方的 IP+端口号存在哪了。
	在客户端发起连接（调用 connect() 函数）时，我们是知道服务器端的 IP+端口号的，应该也是在这时，将本地的 IP+端口号 传给服务器了。但不知道服务器存哪了。
	额，应该是在运输层，更多的细节咱就不知道了。
	额，有套接字五元组的概念：协议，源IP，源端口，目的IP，目的端口。
	额，新理解，客户端中套接字分配的地址，是为服务器分配的，所以在运行多个客户端时，由于服务器都是同一个，所以显示地址信息都一样。那有新问题了，这不就成了，一个套接字对应多个端口了。可套接字和端口是一对一关系。啊，我炸了。



### TCP_NODELAY

#### Nagle算法

不太理解是不是和 停止等待协议和ARQ协议有关


### P154 习题

1. TCP_NODELAY可选项与Nagle算法有关，可通过它禁止Nagle算法。请问何时应考虑禁用Nagle算法？结合收发数据的特性给出说明。

	根据传输数据的特性，网络流量未受太大影响时，不使用Nagle算法要比使用它时传输速度快。例如“传输大文件数据”。将文件数据传入输出缓冲不会花太多时间，因此，即便不使用Nagle算法那，也会在装满输出缓冲时传输数据包。这不仅不会增加数据包的数量，反而会在无需等待ACK的前提下连续传输，因此可以大大提高传输速度。
	
	
	
	
	
## Chapter_10 多进程服务器端

进程，线程，调度，死锁，进程同步，进程互斥，信号量，状态变量……啊，我要死了

### 进程与线程

#### 进程的引入

在多道程序环境下，程序的执行属于并发执行，因此会失去封闭性，并具有间断性和运行结果不可再现性。为了能够使参与并发执行的每个每个程序都能独立地运行，必须为之配备PCB，负责保存和恢复进程运行的现场等任务。

#### 进程的定义

进程是程序的执行过程，是系统进行资源分配和调度（划掉）的一个独立单位。

#### 进程的基本状态

创建，就绪，执行，阻塞，终止

- 创建状态：三个过程：
	- 进程申请PCB，向PCB中填入用于控制和管理进程的信息
	- 为该进程分配运行时所必须的资源
	- 如果资源分配完毕，则转为就绪状态，并插入就绪队列，等待处理机调度为其分配CPU。如果所需资源得不到满足，则创建工作尚未完成，此时进程所处的状态就被称为创建状态。

- 就绪状态：进程已分配到除CPU之外的所有所需资源，只要再获取CPU，便可转为执行状态
- 执行状态：进程获得CPU后，程序处于“正在执行”状态。多处理机系统中，同一时刻的处于正在执行状态的进程不超过处理机数量。
- 阻塞状态：正在执行的进程由于某种原因（如请求资源失败）而暂时无法继续执行，此时会引发进程调度，OS会把处理机分配给其他就绪进程，而让受阻进程处于暂停状态。
- 终止状态：两个步骤：
	- 等待OS善后
	- 将进程的PCB清零，将PCB空间返回给OS
	进入终止状态的进程会变成僵尸进程，直到其他进程完成了对其信息的提取，系统就会删除该进程，即 将PCB清零，并返还OS。
	
#### 进程通信

高级通信机制可归结为4类：共享存储器系统，管道通信系统，消息传递系统以及客户机-服务器系统

#### 线程的引入

为了减少程序在并发执行时所付出的时空开销。在进程切换时，需要保留当前进程的CPU环境，并设置新选中进程的CPU环境，这一过程花费不少处理机时间。因此引入线程，将线程作为调度和分派的基本单位。

#### 线程与进程的比较

1. 调度的基本单位
	
	传统OS中，将进程作为独立调度和分派的基本单位，但每次调度时，都需要进行上下文切换，开销较大。引入线程后，线程作为能独立运行的基本单位，在进行线程切换时（同一进程），仅需保存和设置少量寄存器的内容，切换代价远低于进程。当然，不同进程中的线程进行切换时会引起进程的切换。

2. 并发性

	在引入线程的OS中，不仅进程之间可以并发执行，而且在一个进程中的多个线程之间亦可并发执行。同样，不同进程中的线程也能并发执行。
	
3. 拥有资源

	进程可以拥有资源，可作为系统中拥有资源的一个基本单位。然而，线程可以说是几乎不拥有资源。同一进程的所有线程都具有相同的地址空间，共享进程所拥有的资源。
	
4. 独立性

	同一进程中的不同线程之间的独立性要比不同进程之间的独立性低很多。
	
5. 系统开销

	进程的创建耗时约为线程的30倍，上下文切换耗时约为线程的5倍。

6. 支持多处理机系统

	在多处理机系统中，对于传统的进程，即单线程进程，不管有多少处理机，该进程只能运行在一个处理机上。但对于对于多线程进程，可以将一个进程中的多个线程分配到多个处理机上并行运行。
	
	
### 处理机调度与死锁

在多道程序系统中，调度的实质是一种资源分配，处理机调度是对处理机进行分配。

#### 进程调度任务

- 保存CPU现场信息。进程调度时，需先保存当前进程的CPU现场信息。
- 按某种算法选取进程（调度算法鸽了）。从就绪队列中选取一个进程，将其状态改为运行态。
- 把CPU分配给进程。将PCB内有关CPU的现场信息装入CPU相应的各个寄存器中。

#### 死锁的定义、必要条件与处理方法

##### 死锁定义

如果一组进程中的每个进程，都在等待 仅由该组进程中的其他进程才能引发的 事件发生，那么该组进程是死锁的。

#### 产生死锁的必要条件

- 互斥条件：请求的资源是临界资源。
- 不可抢占条件：进程已获得的资源在未使用完之前不能被抢占，只能在进程使用完后由其自己释放。
- 请求和保持条件：进程请求的资源被其他进程占有导致其被阻塞，同时保持对已分配资源的占有。
- 循环等待条件：形成环了

#### 死锁的处理方法

- 预防死锁
- 避免死锁
- 检测死锁
- 解除死锁

#### 死锁预防

##### 破坏“请求和保持”条件

两种协议：
- 所有进程在开始运行之前，必须一次性地申请所需的全部资源。
- 只有用完全部资源，才能请求新的资源。

##### 破坏“不可抢占”条件

在请求的资源得不到满足而进入阻塞时，必须释放已经占有的所有资源。

##### 破坏“循环等待”条件

对资源进行线性排序，巴拉巴拉

#### 死锁避免

系统处于不安全状态时，可能会进入死锁状态。
避免死锁的实质在于：使系统在进行资源分配使不进入不安全状态。

##### 银行家算法

鸽

#### 死锁的检测与解除

周期性地检测死锁，如果检测到发生死锁，立即利用死锁解除算法。

### 进程同步

#### 概念

异步环境下的一组并发进程因直接制约而相互发送消息、互相合作、互相等待，使得各进程按一定速度执行的过程，称为进程同步

#### 两种形式的制约关系

- 间接相互制约关系（互斥关系）：对临界资源的竞争形成了互斥关系。
- 直接相互制约关系（同步关系）：对临界资源的合作形成了同步关系。

#### 信号量机制

信号量（semaphore）是专门用于解决进程同步与互斥问题的一种通信机制。

##### 整型信号量

wait(S) 和 signal(S) 操作，又被成为 P 操作和 V 操作。

```c
wait(S){
    while(S <= 0) S--;
}

signal(S){
    S++;
}
```

##### 记录型信号量（资源信号量）

整型信号量一直处于“忙等”状态，未遵循“让权等待”原则。所以在记录型信号量中，增加了一个阻塞进程链表。

```c
typedef struct {
    int value;
    struct process_control_block *list;
}semaphore;

wait(semaphore *S){
    S->value--;
    if(S->value < 0) block(S->list);
}

signal(semaphore *S)
{
    if(S->value < 0) wakeup(S->list);
    S->value++;
}

/*
代码理解：
S是临界资源，对临界资源进行封装，value是临界资源的数目，lsit链表用来存储因等待资源而进入阻塞状态的进程。
但是只是这样看，代码是有问题的，要被阻塞/唤醒的进程并未传进来。
*/
```

##### AND 型信号量

前两个信号量，都只是针对一个临界资源，但实际一个进程可能申请多个资源，因此引入 AND 型信号量。

##### 信号量集

前面三个信号量每次只能对某一类临界资源的一个单位进行申请或释放，但可能一次需要N个单位，因此引入信号量集。


#### 生产者-消费者问题

```c
int in = 0, out = 0;
item buffer[n];
semaphore mutex = 1, empty = n, full = 0;

void producer(){
    do{
        produce an item nextp;
        ...
        wait(empty); //必须先对资源信号量进行wait
        wait(mutex);
        
        buffer[in] = nextp;
        in = (in + 1) % n;
        
        signal(mutex);
        signal(full);
        
    }while(true);
}

void consumer(){
    do{
        wait(full);
        wait(mutex);
        
        nextc = buffer[out];
        out = (out + 1) % n;
        
        signal(mutex);
        signal(empty);
        
        sonsume the item int nextc;
        ...
            
    }while(true);
}
```

#### 哲学家进餐问题

```c
semaphore chopstick chopsticks[5] = {1, 1, 1, 1, 1};

do
{
    //think
    
    Swait(chopsticks[(i + 1) % 5], chopsticks[i]);
    
    //eat
    
    Signal(chopsticks[(i + 1) % 5], chopsticks[i]);
    
}while(true);
```


#### 读者-写者问题

```c
/*
问题描述：
	1. 读者与写者不可同时打开同一文件
	2. 多个读者可以同时打开同一文件
	3. 多个写者不可同时打开同一文件

两种解答：
	1. 如果有读者在访问对象，则不管有没有写者在等待，后续读者都可以进行操作
	2. 如果有一个写者在等待访问对象，那么就不会有新读者开始读操作
	
	第一种会导致写者饥饿，第二种会导致读者饥饿
*/

//采用第一种解法
//策略：读时，只有在没有读者时，才 wait(wmutex)
semaphore wmutex = 1, rmutex = 1;//写锁 readcount锁
int readcount = 0;

void reader()
{
    do
    {
        wait(rmutex);
        if(readcount++ == 0) wait(wmutex);
        signal(rmutex);
        
        //reading
        
        wait(rmutex);
        if(--readcount == 0) signal(wmutex);
        signal(rmutex);
        
	}while(true);
}

void writer()
{
    do
    {
        wait(rmutex);
        
        //wirting
        
       	signal(rmutex);
        
	}while(true);
}
```



### fork() wait() waitpid()

```c
#include <unistd.h>

//成功返回进程ID，失败返回 -1，子进程返回 0
pid_t fork(void);
```

```c
#include<sys.wait,h>

/*两个宏：
	WIFEXITED 子进程正常终止时返回 true
	WEXITSTATUS 返回子进程的返回值（exit return）
*/

//statloc 保存子进程退出时的状态信息，存在低八位

//成功时返回终止的子进程ID，失败时返回 -1
pid_t wait(int *statloc);

//使用
int status;

wait(&status);
if(WIFEXITED(status))
{
    puts("Normal termination!");
    printf("Child pass num: %d", WEXITSTATUS(status));
}
```

```c
#include <sys/wait.h>

/*
pid 子进程ID。
	若 pid > 0，只等待这一个子进程
	若 pid = 0，等待同一进程组的所有进程
	若 pid = -1，与 wait() 函数一样，等待所有子进程
	若 pid < -1，等待指定进程组 abs(pid) 的任何子进程
	
statloc 同 wait()
options 传递头文件 sys/wait.h 中声明的常量
	WNOHANG 不阻塞调用 waitpid()
	WUNTRACED 如果子进程暂停执行，则 waitpid() 立即返回
*/

//成功时返回终止的子进程ID（或 0），失败返回 -1
pid_t waitpid(pid_t pid, int *statloc, int options);
```


### 信号

```c
//信号集设定函数
#include <signal.h>

//将指定信号集清 0
int sigemptyset(sigset_t *set);

//将指定信号集置 1
int sigfillset(sigset_t *set);

//将某个信号加入指定信号集
int sigaddset(sigset_t *set, int signum);

//将某个信号从信号集中删除
int sigdelset(sigset_t *set, int signum);

//判断某个信号是否已被加入指定信号集
int sigismember(const sigset_t *set, int signum);
```

```c
#include <signal.h>

/*
函数名 signal
参数 int signo, void (*func)(int)
返回类型 参数类型为int型，返回void型的函数指针
*/

/*
signo 特殊情况信息
	SIGALRM 已到通过调用 alarm 函数注册的时间
	SIGINT 输入 CTRL + C
	SIGCHLD 子进程终止
*/

//为了在产生信号时调用，返回之前注册的函数指针
void (*signal(int signo, void (*func)(int)))(int);
```

```c
#include <unistd.h>

//返回 0 或以秒为单位的距 SIGALRM 信号发生所剩时间 
unsigned int alarm(unisgned int seconds);
```

```c
#include <signal.h>

/*
signum 与 signal 函数相同，传递信号信息
act 对应于第一个参数的信号处理函数
odlact 通过此参数获取之前注册的信号处理函数指针，若不需要则传递 NULL
*/

//成功时返回 0，失败时返回 -1
int sigaction(int signum, const struct sigaction *act,
              struct sigaction *oldact);



struct sigaction {
    void (*sa_handler)(int); //信号处理函数指针
    void (*sa_sigaction)(int, siginfo_t *, void *);
    sigset_t sa_mask; //指定屏蔽的信号
    int sa_flags; //是否使用默认值，默认情况下(sa_flags = 0)，函数会屏蔽自己发送的信号
    void (*sa_restorer)(void);
};
```

### P182 习题

1. 请说明进程变为僵尸进程的过程及预防措施
	僵尸进程是子进程。在子进程结束时，其返回值会传到操作系统，直到返回值被其父进程接收为止，该（子）进程会一直作为僵尸进程存在。所以，为了防止这种情况的发生，父进程必须明确接收子进程结束时的返回值。
	
	
	
	
	
## chapter_11 进程间通信(Inter Process Communication)

### 管道

```c
#include <unistd.h>

//成功返回 0，失败返回 -1
int pipe(int fileds[2]);
```

### 呃啊，其他的以后有需要再学，孩子学不进去了

### P193 习题

1. 什么是进程间通信？分别从概念和内存的角度进行说明

	概括性地说，进程间通信是指两个进程之间交换数据。但是从内存的角度看，可以理解为两个进程共有内存。因为共享的内存区域存在，可以进行数据交换
	
2. 进程间通信需要特殊的IPC机制，这是由操作系统提供的。进程间通信时为何需要操作系统的帮助？

	要想实现IPC机制，需要共享的内存，但由于两个进程之间不共享内存，因此需要操作系统的帮助，也就是说，两进程共享的内存空间必须由操作系统来提供
	
3. “管道”是典型的IPC技术。关于管道，请回答如下问题。
	- 管道是进程间交换数据的路径。如何创建该路径?由谁创建？
		管道是由pipe函数产生的，而实际产生管道的主体是操作系统。
	- 为了完成进程间通信，2个进程需同时连接管道。那2个进程如何连接到同一管道？
		pipe函数通过输入参数返回管道的输入输出文件描述符。这个文件描述符在fork函数中复制到了其子进程，因此，父进程和子进程可以同时访问同一管道。
	- 管道允许进行2个进程间的双向通信。双向通信中需要注意哪些内容？
		管道并不管理进程间的数据通信。因此，如果数据流入管道，任何进程都可以读取数据。因此，要合理安排共享空间的输入和读取。




## chapter_12 I/O 复用

### select

```c
#include <sys/select.h>
#include <sys/time.h>

/*
maxfd 监视对象文件描述符数量 因为我们需要轮询，所以一般设置为 fd + 1
readset 将所有关注“是否存在待读取数据”的文件描述符注册到 readset 中
writeset 将所有关注“是否可传输无阻塞数据”的文件描述符注册到 readset 中
execptset 将所有关注“是否发生异常”的文件描述符注册到 readset 中
timeout 调用 select 后，为防止陷入无限阻塞的状态，超时就返回
*/

//发生错误返回 -1，超时返回 0，关注的事件发生时返回文件描述符
int select(int maxfd, fd_set *readset, fd_set *writeset,
          fd_set *exceptset, const struct timeval *timeout);

FD_ZERO(fd_set *fdset); //将集合 fdset 中的所有位初始化为 0
FD_SET(int fd, fd_set *fdset); //将 fd 注册到 集合 fdset
FD_CLR(int fd, fd_set *fdset); //从集合 fdset 中删除 fd 的信息
FD_ISSET(int fd, fd_set *fdset); //集合 fdset 中的 fd 是否发生变化

struct timeval
{
    long tv_sec; //seconds
    long tv_usec; //microseconds
};
```

### P209 习题

1. 请解释复用技术的通用含义，并说明何为I/O复用。

	复用技术指为了提高物理设备的效率，用最少的物理要素传递最多数据时使用的技术。同样，I/O复用是指将需要I/ O的套接字捆绑在一起，利用最少限度的资源来收发数据的技术
	
2. 多进程并发服务器的缺点有哪些？如何在I/O复用服务器端中弥补？

	多进程并发服务器的服务方式是，每当客户端提出连接要求时，就会追加生成进程。但构建进程是一项非常有负担的工作，因此，向众多客户端提供服务存在一定的局限性。而复用服务器则是将套接字的文件描述符捆绑在一起管理的方式，因此可以一个进程管理所有的I/O操作
	
3. select函数的观察对象中应包含服务器端套接字（监听套接字），那么应将其包含到哪一类监听对象集合？请说明原因

	服务器套接字的作用是监听有无连接请求，即判断接收的连接请求是否存在？应该将其包含到“读”类监听对象的集合中
	
4.  select函数使用的fd_set结构体在Windows和Linux中具有不同的声明。请说明却别，同时解释存在区别的必然性

	Linux的文件描述符从0开始递增，因此可以找出当前文件描述符数量和最后生成的文件描述符之间的关系。但Windows的套接字句柄并非从0开始，并且句柄的整数值之间并无规律可循，因此需要直接保存句柄的数组和记录句柄数的变量
	
	
	
	
## chapter_13 多种 I/O 函数

### send & recv

```c
#include <sys/socket.h>

/*
sockfd	套接字文件描述符
buf		保存待传输数据的缓冲值地址
nbytes	待传输的字节数
flags 	传输数据时指定的可选项信息
*/

//成功时返回发送的字节数，失败时返回 -1
ssize_t send(int sockfd, const void *buf, size_t nbytes, int flags);
```

```c
#include <sys/socket.h>

/*
sockfd	套接字文件描述符
buf		保存接收数据的缓冲值地址
nbytes	可接收的的最大字节数
flags 	接收数据时指定的可选项信息
*/

//成功时返回发送的字节数（收到 EOF 时返回 0），失败时返回 -1
ssize_t recv(int sockfd, const void *buf, size_t nbytes, int flags);
```

<table border="1">
	<caption>send &amp; recv 函数的可选项及含义</caption>
    <tr>
        <th>可选项</th> <th>含义</th>
    </tr>
    <tr>
        <td>MSG_OOB</td> <td>用于传输带外数据（Out-of-band data）</td>
    </tr>
    <tr>
        <td>MSG_PEEK</td> <td>验证输入缓冲中是否存在接收的数据</td>
    </tr>
    <tr>
        <td>MSG_DONTWAIT</td> <td>调用 I/O 函数时不阻塞</td>
    </tr>
    <tr>
        <td>MSG_DONTROUTE</td> <td>数据传输过程中不参照路由表，在本地网络中寻找目的地</td>
    </tr>
    <tr>
        <td>MSG_WAITALL</td> <td>防止函数返回，直到接收全部请求的字节数</td>
    </tr>
    <tr>
        <td>...</td> <td>...</td>
    </tr>
</table>



#### MSG_OOB 发送紧急消息

在 TCP 首部中，URG = 1，紧急指针指向紧急消息距数据报的偏移量（首部大小）。
即使是紧急消息，仍然会按序传输，只不过是在输入缓存中会优先处理。

#### MSG_PEEK & MSG_DONTWAIT 检查输入缓冲

同时设置 `MSG_OEEK` 和 `MSG_DONTWAIT` 选项，以验证输入缓冲中是否存在接受的数据。
设置 `MSG_OEEK` 选项并调用 `recv` 函数，即使读取了输入缓冲的数据也不会删除。


### readv & wtitev

readv & wtitev 函数有助于提高数据通信效率。
`writev` 函数可以将分散保存在多个缓冲中的数据一并发送，
`readv` 函数可以由多个缓冲分别接收。
（人话，可以打包数据）
因此，适当调用这两个函数可以减少 I/O 函数的调用次数

```c
#include <sys/uio.h>

/*
filedes 文件描述符
iov 	iovec 结构体数组的地址值
iovcnt 	第二个参数的数组长度
*/

//成功时返回发送的字节数，失败时返回 -1
ssize_t writev(int filedes, const struct iovec *iov, int iovcnt);
```

```c
struct iovec
{
	void *iov_base; //缓冲地址
    size_t iov_len; //缓冲大小
}
```



### P229 习题

1. 利用 `readv`& `writev` 函数收发数据有何优点？分别从函数调用次数和I/O缓冲的角度给出说明

	`readv`& `writev` 函数可以将分散保存在多个缓冲中的数据一并接受和发送，是对数据进行整合传输及发送的函数，因此可以进行更有效的数据传输。而且，输入输出函数的调用次数也相应减少，也会产生相应的优势
	
2. 通过recv函数见证输入缓冲是否存在数据时（确认后立即返回），如何设置recv函数最后一个参数中的可选项？分别说明各可选项的含义

	同时设置MSG_PEEK选项和MSG_DONTWAIT选项，以验证输入缓冲是否存在可接收的数据。设置MSG_PEEK选项并调用recv函数时，即使读取了输入缓冲数据也不会删除。因此，该选项通常与MSG_DONTWAIT合作，用于调用以非阻塞方式验证待读数据存在与否的函数。

3. 可在Linux平台通过注册时间处理函数接收MSG_OOB数据。那Windows中如何接受？请说明接收方法

	MSG_OOB数据的接收，在select函数中属于异常数据，即在Windows中可以通过异常处理来接收Out-of-band数据
	
	
	
	
	
	
## chapter_14 多播与广播

### 多播

多播（Multicast）的数据传输方式是基于 UDP 完成的。采用多播方式时，可以同时向多个主机传递数据。

#### 多播的数据传输方式及流量方面的优点

- 多播服务器端针对特定多播组，只发送 1 次数据
- 即使只发送一次数据，但该组内的所有用户端都会接收数据
- 多播组数可在 IP 地址范围内任意添加
- 加入特定组即可接收发往该多播组的数据

多播组是 D 类 IP 地址（224.0.0.0 ~ 239.255.255.255）

#### 多播代码总结

- 接收端和发送端都需输入 IP 和 端口号
- 接收方需要调用 bind() ，并且加入多播组
- 发送方可以啥也不干


### 广播

广播只能向同一网络中的主机传输数据

- 直接广播 主机地址全为 1，例如 192.12.34.255
- 本地广播 IP 限定 255.255.255.255

#### 广播代码总结

- 发送方需要输入 IP 和 端口号，接收端只需 端口号
- 发送方需要设置广播选项
- 接收端需要调用 bind()

### P242 习题

1. TTL的含义是什么？请从路由器的角度说明较大的TTL值与较小的TTL值之间的区别及问题

	TTL是Time to Live的缩写，是决定“信息传递距离”的主要因素。TTL表现为整数，没经过一个路由器就减1。如果该值为0，该数据报就会因无法再传递而消失。TTL设置大了会对网络流量造成不良影响，设置太小的话，就可能无法到达目的地
	
2. 多播和广播的异同是什么？请从数据通信的角度进行说明

	多播和广播的相同点是，两者都是以UDP形式传输数据。一次传输数据，可以向两个以上主机传送数据。但传送的范围是不同的：广播是对局域网的广播；而多播是对网络注册机器的多播
	
3. 多播也对网络流量有利，请比较TCP数据交换方式解释其原因。

	多播数据在路由器进行复制。因此，即使主机数量很多，如果各主机存在的相同路径，也可以通过一次数据传输到多台主机上。但TCP无论路径如何，都要根据主机数量进行数据传输。
	
4. 多播方式的数据通信需要MBone虚拟网络。换言之，MBone是用于多播的网络，但它是虚拟网络。请解释此处“虚拟网络”。

	以物理网络为基础，通过软件方法实现的多播通信必备的虚拟网络
	
	
	
	

## chapter_15 套接字和标准 I/O

### 标准 I/O 函数的两个优点

- 具有良好的移植性：函数按照 ANSI C 标准定义
- 可以利用缓冲提高性能：标准I/O带有缓冲区，可以减少系统调用，提高系统效率

### 标准 I/O 函数的几个缺点

- 不容易进行双向通婚
- 有时可能频繁调用 `fflush` 函数
- 需要以 FILE 结构体指针的形式返回文件描述符

### fdopen()

```c
#include <stdio.h>

/*
fd 文件描述符
mode 模式信息 "w","r"...
*/

//成功返回 FILE 结构体指针，失败返回 NULL
FILE *fdopen(int fd, const char *mode);
```

### fileno

```c
#include <stdio.h>

//成功时返回文件描述符，失败返回-1
int fileno(FILE *stream);
```



### P254 习题

1. 请说明标准I/O函数的2个优点。它为何拥有这2个优点？

	基于ANSIX标准具有良好的一致性
	可以利用缓冲提高性能
	
2. 标准IO中，“调用fputs函数传输数据时，调用后应立即开始发送！”，为何这种想法是错误的？为了达到这种效果应添加哪些处理过程？

	通过标准输出函数的传输的数据不直接通过套接字的输出缓冲区发送，而是保存在标准输出函数的缓冲中，然后再用fflush函数进行输出。因此，即使调用“fputs"函数，也不能立即发送数据。如果想保障数据传输的时效性，必须经过fflush函数的调用过程
	
	
	
	
	

## chapter_16 关于 I/O 流分离的其他内容

### 分离“流”的好处

- 在 chapter_10 中，我们通过 `fork()` 复制文件描述符以达到逻辑分离 I/O 流的目的，其目的在于

	- 通过分开输入过程（代码）和输出过程降低实现难度
	- 与输入无关的输出操作可以提高速度

- 在 chapter_15 中，通过 `fdopen()` 创建读/写模式 FILE 指针。分离了输入和输出工具，其目的在于

	- 为了将 FILE 指针按读模式和写模式加以区分
	- 可以通过区分读写模式降低实现难度
	- 通过区分 I/O 缓冲提高缓冲性能

### 分离“流”带来的 EOF 问题

但 通过 `fdopen()` 创建读/写模式 FILE 指针，并没有复制文件描述符，也就是说，两个指针指向的是同一个文件描述符，只要有其中一个指针关闭，文件描述符也就销毁，进而套接字也就关闭了。显然，这就无法实现半关闭效果了。

### `dup()` & `dup2()`

```c
#include <unistd.h>

/*
oldfd 要复制的文件描述符
newfd 给复制后的文件描述符指定一个整数值
*/

//成功返回新文件描述符，失败返回 -1，并设置 errno
int dup(int fd);
int dup2(int oldfd, int newfd);
```

值得注意的是，在 P263 中的 `sep_serv2.c` 程序中，并没有用 `dup` 来复制文件描述符，它通过另一种方式来达到半关闭效果（虽然我看不懂，为什么可以）

```c
shutdown(fileno(writefp), SHUT_WR); // 进入半关闭
fclose(writefp); // 为什么这时候就不会关闭 fd 了
```





### P264 习题

略





## chapter_17 优于 select 的 epoll

### 基于 select 的 I/O 复用技术速度慢的原因

- 调用 select 函数后常见的针对所有文件描述符的循环语句
- 每次调用 select 函数时都需要向该函数传递监视对象信息

select 函数与文件描述符有关，更准确地说，是监视套接字变化的函数。而套接字是由操作系统管理的，所以 select 函数需要把监视对象传递给操作系统，这对程序造成很大负担。

不过 epoll 只在 Linux 下提供支持，不具有兼容性。但大部分操作系统都支持 select 函数

### epoll_create

```c
#include <sys/epoll.h>

/*
size 决定 epoll 例程的容量，不过只是供操作系统参考值
epoll_create 函数创建的资源和套接字一样由操作系统管理，
	返回的文件描述符主要用于区分 epoll例程，
	终止时也要调用 close 函数
*/

//成功时返回 epoll 文件描述符，失败返回 -1
int epoll_create(int size);
```

### epoll_ctl

```c
#include <sys/epoll.h>

/*
epfd 	epoll例程文件描述符
op 		对监视对象的操作，添加、删除或更改
fd		需要监视的对象的文件描述符
event	监视对象的事件类型
*/

//成功返回 0，失败返回 -1
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

#### op

- EPOLL_CTL_ADD   将监视对象注册到 epoll 例程
- EPOLL_CTL_DEL     从 epoll 例程中删除文件描述符
- EPOLL_CTL_MOD	 更改监视对象的关注事件发生情况

向 epoll 例程 A 中注册监视对象 B，主要目的是监视 C 中的事件
```c
epoll_ctl(A, EPOLL_CTL_ADD, B, C)
```


#### event

```c
struct epoll_event
{
    __uint32_t events;
    epoll_data_t data;
}

typedef union epoll_data
{
    void *ptr;
    int fd;
    __unit32_t u32;
    __unit64_t u64;
}epoll_data_t;
```

<table border="1">
	<caption>event.events</caption>
    <tr>
        <th>常量</th> <th>事件</th>
    </tr>
    <tr>
        <td>EPOLLIN</td> <td>需要读取数据的情况</td>
    </tr>
    <tr>
        <td>EPOLLOUT</td> <td>输出缓冲为空，可以立即发送数据的情况</td>
    </tr>
    <tr>
        <td>EPOLLPRI</td> <td>收到 OOB 数据的情况</td>
    </tr>
    <tr>
        <td>EPOLLRDHUP</td> <td>断开连接或半关闭的情况，这在边缘触发方式下非常有用</td>
    </tr>
    <tr>
        <td>EPOLLERR</td> <td>发生错误的情况</td>
    </tr>
    <tr>
        <td>EPOLLET</td> <td>以边缘触发的方式得到事件通知</td>
    </tr>
    <tr>
        <td>EPOLLONESHOT</td> <td>发生一次事件后，相应文件描述符不再收到事件通知，因此需要向 epol_ctl 函数 传递 EPOLL_CTL_MOD，再次设置事件</td>
    </tr>
</table>
 

### epoll_wait

```c
#include <sys/epoll.h>

/*
epfd		epoll 例程的文件描述符
events		用于保存发生事件的文件描述符的集合，空间需动态分配
maxevents	第二个参数的可以保存的最大事件数
timeout		设置超时，单位 ms，传递 -1则一直等待事件发生
*/

//成功监听返回文件描述符数量，超时返回 0，失败返回 -1
int epoll_wait(int epfd, struct epoll_event *events,
               int maxevents, int timeout);
```

### 条件触发和边缘触发

- 条件触发：只要输入缓冲有数据就会一直通知该事件
- 边缘触发：仅输入缓冲收到新数据时注册一次该事件

让我们理一下程序编写演化过程

- epoll 默认条件触发
- 体验条件触发后，为了体验边缘触发，我们将 `BUF_SIZE` 设置为 4
- 因为边缘触发只注册一次事件，所以为了保证读完输入缓冲中的数据，我们应当设置循环读取数据，在读取 `errno` 值后，说明数据已读完，退出循环
- 但 `read()` 在读不到数据时会阻塞，不会返回 -1，因此我们需要将 套接字属性设置为非阻塞

### fcntl

又是你小子，就你功能多，你懂不懂什么叫 单一职责原则 啊
```c
#include <fcntl.h>

//成功返回 cmd 相关参数，失败返回 -1
int fcntl(int filedes, int cmd, ...);
```
将文件改为非阻塞模式：
```c
int flag = fcntl(fd, F_GETFL, 0);
fcntl(fd, F_SETFL, flag | O_NONBLOCK);
```

### P283 习题

1. 利用select函数实现服务器端时，代码层面存在的2个缺点是？

	调用select函数后常见的针对所有文件描述符的循环语句
	每次调用select函数时都需要向该函数传递监视对象信息

2. 无论是select方式还是epoll方式，都需要将监视对象文件描述符信息通过函数调用传递给操作系统。请解释传递该信息的原因

	select和epoll是系统函数，准确地说，是要求观察套接字变化的方式的。套接字是受操作系统进行管理的。既，select和epoll是一个有操作系统执行的函数。因此，应该将监视对象的文件描述符传递给操作系统
	
3. select方式和epoll方式的最大差异在于监视对象文件描述符传递给操作系统的方式。请说明具体的差异，并解释为何存在这种差异。

	epoll不同于select的地方是只要将监视对象文件描述符的信息传递一个给操作系统就可以了。因此epoll方式克服了select方式的缺点，体现在linux内核上保存监视对象信息的方式。
	
4. 虽然epoll是select的改进方式，但select也有自己的缺点。在何种情况下使用select方式更合理

	如果连接服务器的人数不多（不需要高性能），而且需要在多种操作系统（windows和linux）下进行操作，在兼容性方面，使用select会比epoll更合理
	
5. epoll以条件触发或边缘触发方式工作。二者有何区别？从输入缓冲的角度说明这2中方式通知事件的时间点的差异

	在条件触发方式中，只要输入缓冲有数据，就会持续进行事件通知；而在边缘触发中，只有当输入缓冲数据为空时才进行通知

6. 采用边缘触发时可以分离数据的接收和处理时间点。说明 其原因及优点。

	如果使用边缘触发方式，在输入缓冲中接收数据时，只会发生一次事件通知，而且输入缓冲中仍有数据时，不会进行通知，因此可以在数据被接收后，在想要的时间内处理数据。而且，如果分离数据的接收和处理时间点，在服务器中会有更大的灵活性
	
	
	
## chapter_18 多线程服务器端的实现

擦，线程同步好难写啊，简单的 读者-写者问题，花我一整天了。

### 线程和进程的差异

每个进程的内存空间都由

- 保存全局变量的“数据区”
- 向 malloc 等函数的动态分配提供空间的堆（heap）
- 函数运行时使用的栈（stack）

构成

如果以获得多个代码执行流为主要目的，则应该只需分离栈区域，可有如下优势

- 上下文切换时不需要切换数据区和堆
- 可以利用数据区和堆交换数据

这就是线程，多个线程共享数据区和堆。

### 线程的创建和执行流程

#### `pthread_create`

```c
#include <pthread.h>

/*
thread			用来保存创建线程ID的变量地址值
attr			用于传递线程属性的参数
start_routine	执行线程的函数的函数指针
arg				用于向第三个参数传递调用函数时的参数
*/

//成功返回 0，失败返回其他值
int pthread_create(pthread_t *restrict thread,
                   const pthread_attr_t *restrict attr,
                  void *(*start_routine)(void *), void *restrict arg);
```

#### `pthread_join`

```c
#include <pthread.h>

/*
thread	线程ID
status	线程函数返回的指针

调用此函数的进程（或线程）将进入等待状态
*/

//成功返回 0，失败返回其他值
int pthread_join(pthread_t thread, void **status);
```

### 线程同步

线程共享内存空间，因此当访问临界资源时容易引发问题，因此需要解决线程同步问题

#### 互斥量（Mutual Exclusion）

```c
#include <pthread.h>

/*
mutex 	初始化互斥量
attr	互斥量的属性
*/

//成功返回 0，失败返回其他值

int pthread_mutex_init(pthread_mutex_t *mutex,
                       const pthread_mutexattr_t *att);

int pthread_mutex_destory(pthread_mutex_t *mutex);
```

```c
#include <pthread.h>

/*
mutex 	互斥量
*/

//成功返回 0，失败返回其他值

int pthread_mutex_lock(pthread_mutex_t *mutex);

int pthread_mutex_unlock(pthread_mutex_t *mutex);
```


#### 信号量

```c
#include <semaphore.h>

/*
sem		信号量
pshared	可由多个进程共享的信号量
value	信号量初值
*/

//成功返回 0，失败返回其他值
int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_destory(sem_t *sem);
```

```c
#include <semaphore.h>

/*
sem		信号量
*/

//成功返回 0，失败返回其他值
int sem_wait(sem_t *sem);
int sem_post(sem_t *sem);
```


### P312 习题

1. 单CPU系统中如何同时执行多个进程？请解释该过程中发生的上下文切换

	因为系统将CPU切分成多个微小的块后分配给多个进程，为了分时使用CPU,需要”上下文切换“过程。”上下文切换“是指，在CPU改变运行对象的过程中农，执行准备的过程将之前执行的进程数据从换出内存，并将待执行额进程数据传到内存的工作区域
	
2. 为何线程的上下文切换速度相对更快？线程间数据交换为何不需要类似IPC的特别技术？

	因为线程进行上下文切换时不需要切换数据区和堆区。同时，可以利用数据区和堆区进行数据交换
	
3. 请从执行流角度说明进程和线程的区别

	进程：在操作系统中构成单独执行流的单位
	线程：在进程内构成单独执行流的单位

4. 请说明完全销毁Linux线程的两种方法

	`pthread_join` 函数和 `pthread_detach` 函数




## 总结

至此，这本书算是过了一遍了，接下来要学习《LInux高性能服务器编程》。
从 7-21 号 到 8-4 号，共花了 15 天。
本想总结一下，不过，我鸽了。