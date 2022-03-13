# 九、Netty篇

### 1.1. 内核空间、用户空间？

1. 现在操作系统都是采用虚拟存储器，那么对 32 位操作系统而言，它的寻址空间（虚拟存储空间）为 4G（2的32次方）。
2. 操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限，所以，为了保证用户进程不能直接操作内核（kernel），保证内核的安全，操心系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。
3. 针对 linux 操作系统而言，将最高的 1G 字节（从虚拟地址 0xC0000000 到 0xFFFFFFFF），供内核使用，称为内核空间。
4. 而将较低的 3G 字节（从虚拟地址 0x00000000 到 0xBFFFFFFF），供各个进程使用，称为用户空间。

### 1.2. 进程切换？

为了控制进程的执行，内核必须有能力挂起正在 CPU 上运行的进程，并恢复以前挂起的某个进程的执行。这种行为被称为进程切换。因此可以说，任何进程都是在操作系统内核的支持下运行的，是与内核紧密相关的。

从一个进程的运行转到另一个进程上运行，这个过程中经过下面这些变化：

1. 保存处理机上下文，包括程序计数器和其他寄存器。
2. 更新 PCB 信息。
3. 把进程的 PCB 移入相应的队列，如就绪、在某事件阻塞等队列。
4. 选择另一个进程执行，并更新其 PCB。
5. 更新内存管理的数据结构。
6. 恢复处理机上下文。

=> **总而言之就是很耗资源**。

### 1.3. 进程的阻塞？

1. 正在执行的进程，由于期待的某些事件未发生，如请求系统资源失败、等待某种操作的完成、新数据尚未到达或无新工作做等，则由进程自动执行阻塞原语（Block），使自己由运行状态变为阻塞状态。
2. 所以，进程的阻塞，是进程自身的一种**主动行为**，也因此只有处于运行态的进程（已获得 CPU 的进程），才可能将其转为阻塞状态。
3. 当进程进入阻塞状态后，会让出 CPU 资源，不占用 CPU。

### 1.4. 同步、异步、阻塞、非阻塞？

- **同步、异步**：同步和异步，是针对应用程序和内核的**交互**而言的：
  1. 同步，是指用户进程触发 I/O 操作，并等待或者轮询地去查看 I/O 操作是否就绪。
  2. 异步，则是指用户进程触发 I/O 操作以后，便开始做自己的事情，当 I/O 操作已经完成时，会得到操作系统派发的 I/O 操作完成的通知。
- **阻塞、非阻塞**：阻塞和非阻塞，是针对于进程在访问数据时，根据 I/O 操作的就绪状态而采取的不同方式，说白了就是读取或者写入时说调用**函数的不同实现方式**：
  1. 阻塞方式下，读取或者写入函数，应用程序需要一直等待，直到函数执行完成。
  2. 非阻塞方式下，读取或者写入函数，会立即返回一个状态值，相当于是一个尝试性的执行动作。

=> 综上所述，同步、异步是相对于应用和内核的交互方式而言的，同步需要主动去询问，而异步的时候内核在 I/O 事件发生时通知应用程序，而阻塞、非阻塞仅仅是系统在调用系统调用时，函数的实现方式不同而已。

### 1.5. 文件描述符 fd？

1. 文件描述符，File descriptor，是计算机科学中的一个术语，是一个用于表述指向**文件引用**的抽象化概念。
2. 文件描述符，在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开**文件的记录表**：当程序打开一个现有文件，或者创建一个新文件时，内核向进程返回一个文件描述符。
3. 在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展，但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

### 1.6. 缓存 I/O？

1. 缓存 I/O 又被称作标准 I/O，大多数文件系统的默认 I/O 操作都是缓存 I/O。
2. 在 Linux 的缓存 I/O 机制中，操作系统会将 I/O 的数据，缓存在文件系统的页缓存（page cache ）中，也就是说，数据会先被拷贝到操作系统内核的缓冲区，然后才会从操作系统内核的缓冲区中，拷贝到应用程序的地址空间。
3. **缓存 I/O 的缺点**：数据在传输过程中，需要在应用程序地址空间和内核进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是非常大的。

### 1.7. Linux I/O 模式？

对于一次 I/O 访问（以 read 为例），数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间，所以，当一个 read 操作发生时，它会经历两个阶段：

1. 等待数据准备 (Waiting for the data to be ready)
2. 将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)

正式因为这两个阶段，linux 系统产生了下面五种网络模式的方案：
- 阻塞 I/O（blocking I/O）
- 非阻塞 I/O（nonblocking I/O）
- I/O 多路复用（ I/O multiplexing）
- 信号驱动 I/O（ signal driven I/O）
- 异步 I/O（asynchronous I/O）

注：signal driven IO 在实际中并不常用~

#### 1）阻塞 I/O（blocking I/O）

在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概是这样：

![1646817239297](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646817239297.png)

1. 当用户进程调用了 `recvfrom` 这个系统调用，kernel 就开始了 I/O 的第一个阶段：准备数据（对于网络 I/O 来说，很多时候数据在一开始还没有到达，比如，还没有收到一个完整的 UDP 包，这个时候，kernel 就要等待足够的数据到来）。这个过程需要等待，也就是说数据被拷贝到操作系统内核的缓冲区中是需要一个过程的。
2. 而在用户进程这边，整个进程会被阻塞（当然，是进程自己选择的阻塞）。当 kernel 一直等到数据准备好了，它就会将数据从 kernel 中拷贝到用户内存，然后 kernel 返回结果，用户进程才解除 block 状态，重新运行起来。
3. 所以，blocking I/O 的特点是，在 I/O 执行的两个阶段都会被 block 住。

#### 2）非阻塞 I/O（nonblocking I/O）

linux 下，可以通过设置 socket 使其变为 non-blocking，当对一个 non-blocking socket 执行读操作时，流程是这个样子：

![1646817424161](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646817424161.png)

1. 当用户进程发出 read 操作时，如果 kernel 中的数据还没有准备好，那么它并不会 block 用户进程，而是立刻返回一个 error。
2. 从用户进程角度讲 ，它发起一个 read 操作后，并不需要等待，而是马上就得到了一个结果，但用户进程判断结果是一个 error 时，它就知道数据还没有准备好，于是它可以再次发送 read 操作。
3. 一旦 kernel 中的数据准备好了，并且又再次收到了用户进程的 system call，那么它马上就将数据拷贝到了用户内存，然后返回。
4. 所以，nonblocking I/O 的特点是，用户进程需要不断的主动询问 kernel 数据好了没有。

#### 3）I/O 多路复用（ I/O multiplexing）

I/O multiplexing 就是我们说的 `select`、`poll` 和 `epoll`，有些地方也称这种 I/O 方式为事件驱动型 I/O event driven IO。

1. `select` / `epoll` 的好处就在于，单个 process 就可以同时处理**多个**网络连接的 I/O，其基本原理是，`select`、`poll` 、`epoll` 这些 function 会不断的轮询所负责的所有 socket，当某个 socket 有数据到达了，就通知用户进程。
2. 当用户进程调用了 `select`，那么整个进程会被block，同时，kernel会“监视”所有 `select` 负责的socket，当任何一个 socket 中的数据准备好了，select 就会返回。这个时候用户进程再调用 read 操作，将数据从 kernel 拷贝到用户进程。
3. 所以，I/O 多路复用的特点是，通过一种机制，让一个进程能同时等待多个文件描述符，而这些文件描述符（套接字描述符）其中的任意一个进入读就绪状态，`select` 函数就可以返回了。

![1646817566451](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646817566451.png)

- 这个图和 blocking IO 的图其实并没有太大的不同，事实上，还更差一些，因为这里需要使用两个 system call  (`select` 和 `recvfrom`)，而blocking IO只调用了一个 system call (`recvfrom`)。
- 但是，用 `select` 的优势在于它可以同时处理多个 connection，所以，如果处理的连接数不是很高的话，使用 `select` / `epoll` 的web server不一定比使用 multi-threading + blocking IO 的 web server 性能更好，可能延迟还更大，即其优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。
- 在 IO multiplexing Model 当前模型中，实际中，对于每一个 socket，一般都设置成为 non-blocking，但是，如上图所示，整个用户的 process 其实是一直被 block 的，只不过 process 过程是被 `select` 这个函数 block，而不是被 socket I/O 给 block 住。

=> 所以，基于多路复用模型实现的代码，处理网络 I/O 过程，可以看作是同步非阻塞的。

#### 4）信号驱动 I/O（ signal driven I/O）

1. 在信号驱动式 I/O 模型中，应用程序使用套接口进行信号驱动 I/O，并安装一个信号处理函数，进程继续运行并不阻塞。
2. 当数据准备好时，进程会收到一个 SIGIO 信号，可以在信号处理函数中，调用 I/O 操作函数处理数据。

![1646818238023](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646818238023.png)

- **优点**：线程并没有在等待数据时被阻塞，可以提高资源的利用率。
- **缺点**：信号 I/O 在大量 I/O 操作时，可能会因为信号队列溢出，导致没法通知。
- **适用场景**：
  1. 信号驱动 I/O 尽管对于处理 UDP 套接字来说有用，即这种信号通知意味着到达一个数据报，或者返回一个异步错误。
  2. 但是，对于 TCP 而言，信号驱动的 I/O 方式近乎无用，因为导致这种通知的条件为数众多，每一个来进行判别会消耗很大资源，与前几种方式相比优势尽失。

#### 5）异步 I/O（asynchronous I/O）

inux 下的 asynchronous I/O 其实用得很少，先看一下它的流程：

![1646818029954](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646818029954.png)

1. 用户进程发起 read 操作之后，立刻就可以开始去做其它的事。
2. 而另一方面，从 kernel 的角度，当它受到一个 asynchronous read 之后，首先它会立刻返回，不会对用户进程产生任何 block。
3. 然后，kernel 会等待数据准备完成后，再将数据拷贝到用户内存，当这一切都完成之后，kernel 会给用户进程发送一个 signal，告诉它 read 操作完成了。

### 1.8. select、poll、epoll？

1. `select`、`poll`、`epoll` 都是 I/O 多路复用的机制。
2. I/O多路复用，就是通过一种机制，让一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。
3. 但 `select`、`poll`、`epoll` 本质上都是同步 I/O，因为它们都需要在读写事件就绪后，自己负责进行读写，也就是说这个读写过程是阻塞的，而异步 I/O 则无需自己负责进行读写，操作系统会负责把数据从内核拷贝到用户空间。

#### 1）select

```c++
int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

1. `select` 函数监视的文件描述符分 3 类，分别是 `writefds`、`readfds`、和 `exceptfds`。
2. 调用后，`select` 函数会阻塞，直到有描述副就绪（比如有数据 可读、可写、或者有 except），或者超时（`timeout` 指定等待时间，如果立即返回设为 null 即可），则函数返回。
3. 当 `select` 函数返回后，可以通过遍历 `fdset` ，来找到就绪的描述符。

- **优点**：`select` 目前几乎在所有的平台上支持，其良好跨平台支持也是它的一个优点。
- **缺点**：`select` 的一个缺点在于，单个进程能够监视的文件描述符的数量存在最大限制，在 Linux 上一般为1024，可以通过修改宏定义，甚至重新编译内核的方式来提升这一限制，但是这样也会造成效率的降低。

#### 2）poll

```c++
int poll (struct pollfd *fds, unsigned int nfds, int timeout);

struct pollfd {
    int fd; /* file descriptor */
    short events; /* requested events to watch */
    short revents; /* returned events witnessed */
};
```

- **特点**：
  1. 不同与 `select` 使用三个位图来表示三个 `fdset` 的方式，`poll` 只使用一个 `pollfd` 的指针来实现。
  2. `pollfd` 结构包含了要监视的 event，和 发生的 event，不再使用 `select` “参数-值” 的传递方式。
  3. 同时，`pollfd` 并没有最大数量限制，但数量过大后性能也还是会下降。 
  4. 和 `select` 函数一样，`poll` 返回后，需要轮询 `pollfd` 来获取就绪的描述符。

- **缺点**：
  1. 从上面看，`select` 和 `poll` 都需要在返回后，通过遍历文件描述符来获取已经就绪的 socket。
  2. 事实上，同时连接的大量客户端，在一时刻可能只有很少的处于就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降。

#### 3）epoll

`epoll` 是在 Linux 2.6 内核中提出的，是之前的 `select` 和`poll` 的增强版本。

1. 相对于 `select` 和 `poll` 来说，`epoll` 更加灵活，没有描述符限制。
2. `epoll` 使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件，存放到内核的一个事件表中，这样在用户空间和内核空间只需一次的 copy。

```c++
int epoll_create(int size)；// 创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

##### API  - int epoll_create(int size)

1. 创建一个 `epoll` 句柄，size 用来告诉内核这个监听的数目一共有多大，这个参数不同于 `select()` 中的第一个参数，给出最大监听的 fd+1 的值，参数 size 并不是限制了 `epoll` 所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议。
2. 当创建好 `epoll` 句柄后，它就会占用一个 fd 值，在 linux 下如果查看 `/proc/进程id/fd/`，是能够看到这个fd 的，所以在使用完 `epoll` 后，必须调用 `close()` 关闭，否则可能导致 fd 被耗尽。

##### API - int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)

此函数是对指定描述符 fd 执行 op 操作：

1. **epfd**：是 `epoll_create()` 的返回值。
2. **op**：表示op操作，用三个宏来表示：
   - 1）`EPOLL_CTL_ADD` ：添加对 fd 的监听事件。
   - 2）`EPOLL_CTL_DEL`：删除对 fd 的监听事件。
   - 3）`EPOLL_CTL_MOD`：修改对 fd 的监听事件。
3. **fd**：是需要监听的文件描述符 fd。
4. **epoll_event**：是告诉内核需要监听什么事，`struct epoll_event` 结构如下：

```c++
// struct epoll_event结构
struct epoll_event {
  __uint32_t events;  /* Epoll events */
  epoll_data_t data;  /* User data variable */
};

//events可以是以下几个宏的集合：
EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
EPOLLOUT：表示对应的文件描述符可以写；
EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
EPOLLERR：表示对应的文件描述符发生错误；
EPOLLHUP：表示对应的文件描述符被挂断；
EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里
```

##### API - int epoll_wait(int epfd, struct epoll_event \* events, int maxevents, int timeout)

等待 `epfd` 上的 I/O 事件，最多返回 `maxevents` 个事件。

1. 参数 `events` 用来从内核得到事件的集合。
2. 参数 `maxevents` 用来告诉内核这个 `events` 有多大，这个 `maxevents` 的值不能大于创建 `epoll_create()` 时的size。
3. 参数 `timeout` 是超时时间，毫秒，0 会立即返回，-1 表示不确定，也有说法说是永久阻塞。
4. **返回值**：该函数返回需要处理的事件数目，如返回0表示已超时。

###### epoll 工作模式

`epoll` 对文件描述符的操作有两种模式：

- **LT 模式**：level trigger，水平触发模式，默认模式。

  1. 当 `epoll_wait` 检测到描述符事件发生，并将此事件通知应用程序，应用程序可以不用立即处理该事件。
  2. 在下次调用 `epoll_wait` 时，操作系统会再次响应应用程序并通知此事件

- **ET模式**：edge trigger，边缘触发模式。

  1. 当 `epoll_wait` 检测到描述符事件发生，并将此事件通知应用程序，应用程序必须立即处理该事件。
  2. 如果一直不对这个 `fd` 做 I/O 操作，内核不会发送更多的通知 (only once)，在下次调用 `epoll_wait` 时，操作系统不会再次响应应用程序通知此事件。

  => **优点**：ET 模式在很大程度上，减少了 `epoll` 事件被重复触发的次数，因此效率要比 LT 模式高。

  => **局限**：`epoll` 工作在 ET 模式时，必须使用 no-block socket 非阻塞套接口，以避免由于一个文件句柄的阻塞读、或者阻塞写操作，把处理多个文件描述符的任务饿死。

##### 4）epoll 总结

- **原理**：
  1. 在 `select` / `poll` 中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描。
  2. 而 `epoll` 事先通过 `epoll_ctl()` 来注册一个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似 `callback` 的回调机制，迅速激活这个文件描述符，当进程调用 `epoll_wait()` 时便得到通知。
  3. 这样，`epoll` 去掉了遍历文件描述符，而是通过监听回调的的机制，这也正是 `epoll` 的魅力所在。
- **优点**：
  1. **监视的描述符数量不受限制**：`epoll` 所支持的 fd 上限，是最大可以打开文件的数目，这个数字一般远大于 2048。比如，在 1GB 内存的机器上大约为10 万左右，具体数目可以 `cat /proc/sys/fs/file-max` 察看，一般来说这个数目和系统内存关系很大。
     - `select` 的最大缺点就是，进程打开的 fd 是有数量限制的，这对于连接数量比较大的服务器来说，根本不能满足。虽然也可以选择多进程的解决方案(Apache就是这样实现的)，不过虽然 linux 上面创建进程的代价比较小，但仍旧是不可忽视的，加上进程间数据同步远比不上线程间同步的高效，所以也不是一种完美的方案。
  2. **I/O 效率不会随着监视 fd 的数量的增长而下降**：`epoll` 不同于 `select` 和 `poll` 轮询的方式，而是通过每个 fd 定义的回调函数来实现的，只有就绪的 fd 才会执行回调函数。

=> 如果没有大量的 idle -connection 或者 dead-connection，`epoll` 的效率并不会比 `select` / `poll` 高很多，但是当遇到大量的 idle- connection，就会发现 `epoll` 的效率大大高于 `select` / `poll`。

### 1.9. Java 复制？

#### 1）直接赋值

直接赋值， 比如在 Java 中，`A a1 = a2`，实际上复制的是引用，也就是说 `a1` 和 `a2` 指向的是同一个对象，因此，当 `a1` 变化时，`a2` 里面的成员变量也会跟着变化。

#### 2）浅复制

浅复制，指创建一个新对象时，把对象的非静态字段复制到新对象上，如果字段是值类型，则会对该字段执行复制，如果字段是引用类型，则只会复制引用不复制引用的对象，因此，原始对象及其复制后的副本，它们的引用类型属性引用属于同一个对象。

=> 通过复制引用的方式，来完成对象引用类型属性值的复制，叫做浅复制。

- **实现方式**：实现 `Cloneable` 接口，然后调用 `Object#clone()`。

  ```java
  @Test
  public void testBook() {
      Chapter chapter1 = new Chapter("第一章", 2500, 15);
      Chapter chapter2 = new Chapter("第二章", 2600, 16);
      Book book = new Book(1l, "三国演义", "罗贯中", Lists.newArrayList(chapter1, chapter2));
      
      Book cloneBook = (Book) book.clone();
      
      // false
      System.out.println(book == cloneBook);  
      // true
      System.out.println(book.getChapterList() == cloneBook.getChapterList()); 
   
      //给book对象的chapterList加一个元素，可以看到cloneBook的chapter也变化了
      book.getChapterList().add(new Chapter("第三章", 2500, 15));
      System.out.println(cloneBook.getChapterList().size()); //3
  }
  ```

#### 3）深复制

深拷贝，指不仅复制对象本身，还复制其引用类型属性所指向的对象。

- **实现方式**： 实现 `Serializable` 接口，把对象写到一个流里，再从流里读出来，可以重建对象。

  ```java
  @Test
  public void testDeepClone() throws Exception{
      Chapter chapter1 = new Chapter("第一章", 2500, 15);
      Chapter chapter2 = new Chapter("第二章", 2600, 16);
      Book book = new Book(1l, "三国演义", "罗贯中", Lists.newArrayList(chapter1, chapter2));
      
      //将对象转换为字节流写入流中
      ByteArrayOutputStream baos = new ByteArrayOutputStream();
      ObjectOutputStream oos = new ObjectOutputStream(baos);
      oos.writeObject(book);
   
      //从流里读出来
      ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
      ObjectInputStream ois = new ObjectInputStream(bais);
      Book cloneBook = (Book) ois.readObject();
      
      //关闭流
      baos.close();
      oos.close();
      bais.close();
      ois.close();
   
      /**
       * 此处如果没有实现Serializable接口，就会报错java.io.NotSerializableException
       * Library持有任何对象的类型都要实现Serializable接口，否则会报错
       */
      // false
      System.out.println(book == cloneBook);
      // false
      System.out.println(book.getChapterList() == cloneBook.getChapterList());
  }
  ```

### 2.0. Java 序列化？

- **序列化**：

  - 指将 Java 对象转化为字节序列的过程，即将对象的状态转化成字节流，然后可以通过这些值再生成相同状态的对象。
  - 对象序列化，是对象**持久化**的一种实现方法，是将对象的属性和方法转化为一种序列化的形式，用于存储和传输。

- **反序列化**：指将字节序列转化为 Java 对象的过程，即将对象字节序列重建对象的过程。

- **优点**：

  - 实现了数据的持久化，通过序列化可以把数据永久地保存到硬盘上，通常是放在文件里，比如 Redis 的RDB。
  - 利用序列化实现远程通信，即在网络上传送对象的字节序列，比如 Google 的 ProtoBuf。

- **反序列失败场景**：

  - 如果 `serialVersionUID` 不一致，会导致反序列化失败。
  - **`serialVersionUID` 作用**：
    1. 如果用户没有自己声明一个 `serialVersionUID`，接口会默认生成一个 `serialVersionUID`。
    2. 但是，强烈建议用户自定义一个 `serialVersionUID`，因为默认的 `serialVersinUID` 对于 class 的细节非常敏感，反序列化时可能会导致 `InvalidClassException` 异常。
    3. 比如说先进行序列化，然后在反序列化之前修改了类，那么就会报错，因为修改了类，对应的`serialversionUID` 也变化了，而序列化和反序列化就是通过对比其 `serialversionUID` 来进行的，一旦 `serialversionUID` 不匹配，反序列化就无法成功。

  ```java
  private static final Long serialVersionUID = 1573531233370L;
  ```

- **transient  关键字**：可以阻止所修饰的变量被序列化到文件中。 
  1. 在变量前加上 `transient`  关键字，可以阻止该变量被序列化到文件中，在被反序列
     化后，`transient` 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。 
  2. 服务器端给客户端发送序列化对象数据，对象中有一些数据是敏感的，比如密码字符串
     等，希望对该密码字段在序列化时，进行加密，而客户端如果拥有解密的密钥，只有在
     客户端进行反序列化时，才可以对密码进行读取，这样可以一定程度保证序列化对象的
     数据安全。

### 2.1. 什么是 Java I/O？

- Java#I/O，是以流为基础进行数据输入输出的，所有数据被串行化出写入输出流，所谓串行化，就是数据按顺序进行输入输，简单来说就是 Java 通过 IO 流的方式和外部设备进行交互。

- 在 Java 类库中，IO 部分的内容是很庞大的，因为它涉及的领域很广泛：标准输入输出、文件的操作、网络数据传输流、字符串流、对象流等等。
- 比如程序从服务器上下载图片，就是通过流的方式，从网络上以流的方式加载到程序中，最后保存到硬盘中。

### 2.2. Java I/O 流分类？

![1646819558347](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646819558347.png)

#### 1）按照实际IO操作来分

1. **输入流**：从文件读入到内存，只能进行读操作。
2. **输出流**：从内存读出到文件，只能进行写操作。

=> **注意**：输出流可以帮助我们创建文件，而输入流不会。

#### 2）按照读写的单位大小来分

1. **字节流**：以字节为单位，每次次读入或读出是 8 位数据（一个字节占 8 bit），可以读任何类型数据，比如图片、文件、音乐视频等，在 Java 代码中，接收数据只能为 `byte[]` 。
2. **字符流**：以字符为单位，每次次读入或读出是 16 位数据（char 占两个字节），其只能读取字符类型数据，在 Java 代码中，接收数据为一般为 `char[]`。

#### 3）按照读写时是否直接与硬盘、内存等节点连接来分

1. **节点流**：直接与数据源相连，可读入或读出。
2. **处理流**：也叫**包装流**，是对一个对于已存在的流连接进行包装，通过所封装流的功能调用，以实现数据读写，比如添加个 Buffered 缓冲区的 `BufferedInputStream`。

=> **注意**：为什么要有处理流？主要作用是在读入或写出时，对数据进行缓存，以减少 I/O 次数，以便下次更好更快地读写文件，所以有了处理流。

### 2.3. Java I/O 常用实现类？

![1646820304336](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646820304336.png)

#### 1）FileReader | 字符节点输入流

```java
public class FileReaderTest {

    public static void main(String[] args) {
        FileReaderTest fileReaderTest = new FileReaderTest();
        fileReaderTest.printStr(Constant.FILE_READ_PATH);
    }

    public void printStr(String path) {
        // 读取文件
        FileReader fileReader = null;
        try {
            fileReader = new FileReader(path);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            return;
        }

        // 读取文件内容
        int res = -1;
        List<Character> characterList = new ArrayList<>();
        while (true) {
            try {
                res = fileReader.read();
                if(res == -1) {
                    break;
                }
            } catch (IOException e) {
                e.printStackTrace();
                break;
            }

            characterList.add((char) res);
        }

        // 打印文件内容
        StringBuilder sb = new StringBuilder();
        for (Character character : characterList) {
            if(character != '\r' && character != '\n') {
                sb.append(character);
            }
            if(character == '\n') {
                // HelloWorld!
                // Nice!
                // 哈喽!
                System.err.println(sb.toString());
                sb = new StringBuilder();
            }
        }

        // 关闭流
        try {
            fileReader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 2）BufferedReader | 字符处理输入流

```java
public class BufferedReaderTest {

    public static void main(String[] args) {
        BufferedReaderTest bufferedReaderTest = new BufferedReaderTest();
        bufferedReaderTest.printStr(Constant.FILE_READ_PATH);
    }

    public void printStr(String path) {
        // 读取文件
        FileReader fileReader = null;
        try {
            fileReader = new FileReader(path);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            return;
        }

        // 节点流对接处理流: 缓冲区的作用的主要目的是, 避免每次和硬盘打交道，提高数据访问的效率
        BufferedReader bufferedReader = new BufferedReader(fileReader);

        // 读取文件内容
        String str = null;
        List<String> stringList = new ArrayList<>();
        while (true) {
            try {
                str = bufferedReader.readLine();
                if(str == null) {
                    break;
                }
            } catch (IOException e) {
                e.printStackTrace();
                break;
            }

            stringList.add(str);
        }

        // 打印文件内容
        // [HelloWorld!, Nice!, 哈喽!]
        System.err.println(stringList);

        // 关闭流
        try {
            bufferedReader.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fileReader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### 3）FileWriter | 字符节点输出流

```java
public class FileWriterTest {

    public static void main(String[] args) {
        // 读取文件
        FileWriter fileWriter = null;

        try {
            // 文件不存在时则创建, true代表追加式写入
            fileWriter = new FileWriter(Constant.FILE_WRITE_PATH, true);
        } catch (IOException e) {
            e.printStackTrace();
            return;
        }

        // 写入内容到文件
        try {
            fileWriter.write("哈喽Write!\r\n");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 关闭流
        try {
            fileWriter.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 读取并打印文件内容: [哈喽Write!, 哈喽Write!]
        BufferedReaderTest bufferedReaderTest = new BufferedReaderTest();
        bufferedReaderTest.printStr(Constant.FILE_WRITE_PATH);
    }
}
```

#### 4）BufferedWriter | 字符处理输出流

```java
public class BufferedWriterTest {

    public static void main(String[] args) {
        // 读取文件
        FileWriter fileWriter = null;

        try {
            // 文件不存在时则创建, true代表追加式写入
            fileWriter = new FileWriter(Constant.FILE_WRITE_PATH, true);
        } catch (IOException e) {
            e.printStackTrace();
            return;
        }

        // 节点流对接处理流: 缓冲区的作用的主要目的是, 避免每次和硬盘打交道，提高数据访问的效率
        BufferedWriter bufferedWriter = new BufferedWriter(fileWriter);

        // 写入内容到文件
        try {
            bufferedWriter.write("哈喽BufferdWrite!\r\n");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 关闭流
        try {
            bufferedWriter.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fileWriter.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        // 读取并打印文件内容: [哈喽BufferdWrite!, 哈喽BufferdWrite!]
        BufferedReaderTest bufferedReaderTest = new BufferedReaderTest();
        bufferedReaderTest.printStr(Constant.FILE_WRITE_PATH);
    }
}
```

#### 5）FileInputStream、FileOutputStream | 字节节点输入输出流

```java
public class FileInputOutTest {

    public static void main(String[] args) {
        // 读取文件
        FileInputStream fileInputStream = null;
        FileOutputStream fileOutputStream = null;
        try {
            fileInputStream = new FileInputStream(Constant.FILE_READ_PATH);

            // 文件不存在时则创建, true代表追加式写入
            fileOutputStream = new FileOutputStream(Constant.FILE_WRITE_PATH, true);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            return;
        }

        try {
            // 读取可用空间
            byte[] bytes = new byte[fileInputStream.available()];

            // 读取流内容到bytes数组中
            fileInputStream.read(bytes);

            // 输出bytes数组到另一个文件中
            fileOutputStream.write(bytes);
        } catch (IOException e) {
            e.printStackTrace();
            return;
        } finally {
            try {
                fileOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        // 打印并读取输入后的文件结果: [哈喽BufferdWrite!, 哈喽BufferdWrite!, HelloWorld!, Nice!, 哈喽!]
        BufferedReaderTest bufferedReaderTest = new BufferedReaderTest();
        bufferedReaderTest.printStr(Constant.FILE_WRITE_PATH);
    }
}
```

#### 6）BufferedInputStream、BufferedOutputStream | 字节处理输入输出流

```java
public class FileBufferdInputOutTest {

    public static void main(String[] args) {
        // 读取文件
        FileInputStream fileInputStream = null;
        FileOutputStream fileOutputStream = null;
        try {
            fileInputStream = new FileInputStream(Constant.FILE_READ_PATH);

            // 文件不存在时则创建, true代表追加式写入
            fileOutputStream = new FileOutputStream(Constant.FILE_WRITE_PATH, true);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            return;
        }

        // 节点流对接处理流: 缓冲区的作用的主要目的是, 避免每次和硬盘打交道，提高数据访问的效率
        BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
        BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);

        while (true) {
            try {
                int read = bufferedInputStream.read();
                if(read == -1) {
                    break;
                }

                bufferedOutputStream.write(read);
            } catch (IOException e) {
                e.printStackTrace();
                break;
            }
        }

        try {
            bufferedOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                bufferedInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    fileOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    try {
                        fileInputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }

        // 打印并读取输入后的文件结果: [哈喽BufferdWrite!, 哈喽BufferdWrite!, HelloWorld!, Nice!, 哈喽!, HelloWorld!, Nice!, 哈喽!, HelloWorld!, Nice!, 哈喽!]
        BufferedReaderTest bufferedReaderTest = new BufferedReaderTest();
        bufferedReaderTest.printStr(Constant.FILE_WRITE_PATH);
    }
}
```

### 2.4. TCP Socket API？

![1646914477436](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646914477436.png)

#### 1）服务端 - API

1. new Socket（..）：构造 Socket 对象，在 Java 中，为 `new ServerSocket() `。
2. bind（..）：绑定端口。
3. listen（）：监听端口，在 Java 中，该方法已经与 `bind（..）` 方法融为一体了。
4. accept（）：TCP 三次握手成功，则唤醒当前阻塞的线程，在 Java 中，返回一个 `Socket` 对象。
5. read（） / write（）：在 Java 中，是通过使用 `Socket` 的输入输出流来实现读取和写出。
6. close（）：关闭套接字文件描述符。

#### 2）客户端 - API

1. new Socket（..）：构造 Socket 对象，在 Java 中，为 `new Socket() ` 。
2. connect（）：连接目标服务端口，在 Java 中，该方法已经与`new Socket() ` 方法融为一体了。
3. read（） / write（）：在 Java 中，是通过使用 `Socket` 的输入输出流来实现读取和写出。
4. close（）：关闭套接字文件描述符。



### 2.5. Java BIO、NIO、AIO？

#### 1）BIO

BIO，**同步阻塞 I/O**：

1. 服务器实现一个连接一个线程，即客户端有连接请求时，服务器端就需要启动一个线程进行处理，没处理完之前，此线程不能做其他操作。
2. 如果是单线程的情况下，传输的文件很大，可以通过线程池机制改善。

=> 因此，BIO 适用于连接数目比较小、且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，是 JDK 1.4 以前的唯一选择，同时程序也直观简单、易于理解。

##### BIO Server

```java
public class BioServerTest {

    private static final AtomicInteger COUNT = new AtomicInteger(0);

    public static void start(int port) throws IOException {
        // socket
        ServerSocket server = new ServerSocket();
        // bind & listen
        server.bind(new InetSocketAddress(port));
        System.err.println("bind & listen!");

        while (true) {
            // accept
            final Socket socket = server.accept();// block!
            int count = COUNT.incrementAndGet();
            System.err.println("accept! ip=" + socket.getRemoteSocketAddress() + ", count=" + count);

            // or user thread pool
            new Thread(() -> {
                try {
                    BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                    PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
                    String line = in.readLine();

                    while (line != null) {
                        System.err.println("read: " + line);
                        out.println(line);
                        out.flush();
                        line = in.readLine();
                    }
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                    try {
                        socket.close();
                    } catch (IOException ee) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }

    }

    public static void main(String[] args) throws IOException {
        start(8084);
    }
}
```

##### BIO Client

```java
public class BioClientTest implements Cloneable {

    public static final String[] commands = new String[]{
            "hi\n",
            "i am client\n",
            "helloworld\n",
            "java and netty\n"
    };

    public static void main(String[] args) throws IOException {
        BioClientTest bioClientTest = new BioClientTest();
        try {
            Object clone = bioClientTest.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }

        int concurrent = 100;
        Runnable task = () -> {
            try {
                Socket socket = new Socket("127.0.0.1", 8084);
                DataOutputStream out = new DataOutputStream(socket.getOutputStream());
                for (String str : commands) {
                    out.write(str.getBytes());
                }
                out.flush();

                Thread.sleep(100);
                BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                while (br.ready()) {
                    Thread.sleep(100);
                    System.out.println(br.readLine());
                }

                socket.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        };

        ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 2, 60L, TimeUnit.SECONDS, new LinkedBlockingQueue<>(1024));
        for (int i = 0; i < concurrent; i++) {
            executor.execute(task);
        }
        executor.shutdown();
    }
}
```

#### 2）NIO

NIO，**同步非阻塞 I/O**：

1. 服务器实现一个连接一个线程，即客户端发送的连接请求都会注册到多路复用器上。
2. 多路复用器轮询到连接有 I/O 请求时，才会启动一个线程进行处理。

=> 因此，NIO 方式适用于连接数目多、且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，在 JDK 1.4 之后开始支持。

##### NIO Server

Selector 可以处理 4 个事件：

1. **OP_READ = 1**：读取事件。
2. **OP_WRITE = 4**：写出事件。
3. **OP_CONNECT = 8**：通道已连接事件。
4. **OP_ACCEPT = 16**：服务端套接字已准备就绪事件。

```java
public class NioServerTest {

    public static void start(int port) throws IOException {
        // non blocking
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        serverChannel.configureBlocking(false);

        // bind & listen
        InetSocketAddress address = new InetSocketAddress(port);
        serverChannel.bind(address);
        System.err.println("bind & listen");

        Selector selector = Selector.open();
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.err.println("注册事件: OP_ACCEPT");

        while (true) {
            // epoll: blocking
            selector.select();

            // OP_ACCEPT | OP_READ
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            Iterator<SelectionKey> it = readyKeys.iterator();
            while (it.hasNext()) {
                SelectionKey key = it.next();

                // OP_ACCEPT: 三次握手成功, 注册读事件
                if (key.isAcceptable()) {
                    ServerSocketChannel server = (ServerSocketChannel) key.channel();
                    SocketChannel socket = server.accept();
                    System.err.println("Accept !");

                    socket.configureBlocking(false);
                    socket.register(selector, SelectionKey.OP_READ);
                    System.err.println("注册事件: OP_READ");
                } else if (key.isReadable()) {
                    SocketChannel socket = (SocketChannel) key.channel();
                    final ByteBuffer buffer = ByteBuffer.allocate(64);
                    final int bytesRead = socket.read(buffer);
                    if (bytesRead > 0) {
                        System.err.println("read: " + new String(buffer.array()).trim());
                        buffer.flip();
                        int ret = socket.write(buffer);
                        // 尝试写, 没写完则要注册写事件
//                         if (ret <=0) {
//                        	 //register op_write
//                         }
                        buffer.clear();
                    } else if (bytesRead < 0) {
                        key.cancel();
                        socket.close();
                        System.err.println("Client close");
                    }
                }

                // 从readyKeys中移除, 免得下次重新执行
                it.remove();
            }
        }
    }


    public static void main(String[] args) throws InterruptedException, IOException {
        start(8084);
    }
}	
```

#### 3）AIO

AIO，**异步非阻塞 I/O**：

1. 服务器实现模式为一个有效请求一个线程。
2. 客户端的 I/O 请求都是由操作系统先完成了，再通过回调，通知客户端应用去启动线程进行处理。

=> AIO 方式适用于连接数目多、且连接比较长（重操作）的架构，比如相册服务器，可以充分调用操作系统参与并发操作，编程比较复杂，在 JDK 1.7 之后开始支持。

- AIO 属于 NIO 包中的类实现，其实 I/O 主要分为 BIO和 NIO，AIO 只是附加品，主要是为了解决 I/O 不能异步实现的问题。
- 在以前很少有 Linux 系统支持 AIO，Windows 的 IOCP 就是此 AIO 模型，但是现在的服务器一般都是支持AIO 操作。

#### 4）总结

1. BIO 是阻塞的，NIO 是非阻塞的。
2. BIO 是面向流的，只能单向读写，NIO 是面向缓冲的, 可以双向读写。
   - 使用 BIO 做 Socket 连接时，由于单向读写，当没有数据时，会挂起当前线程，阻塞等待，为防止影响其它连接,需要为每个连接新建线程处理，然而系统资源是有限的，不能过多的新建线程，线程过多带来线程上下文的切换，从来带来更大的性能损耗，因此需要使用 NIO 进行 BIO 多路复用，使用一个线程来监听所有 Socket连接，使用本线程或者其他线程处理连接。
3. AIO 以非阻塞的异步方式发起 I/O 操作，当 I/O 操作进行后，自己可以去做其他操作，由操作系统内核空间提醒 I/O 操作已完成。

### 2.6. Java IO 设计模式？

在高性能 I/O 设计中，有两个比较著名的模式，Reactor 和 Proactor 模式，其中 Reactor 模式用于同步 I/O，而 Proactor 运用于异步 I/O 操作。

#### Reactor 模式

Reactor 模式，应用于同步 I/O 场景，以 read 操作为例，Reactor 中的具体步骤为：

1. 应用程序注册 `读就绪` 事件和相关联的事件处理器。
2. 事件分离器等待事件的发生。
3. 当发生 `读就绪` 事件时，事件分离器调用第一步注册的事件处理器
4. 事件处理器首先执行实际的读取操作，然后根据读取到的内容进行进一步的处理。

![1646831826891](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646831826891.png)

##### 1）单 Reactor 单线程模式

###### Reactor NIO 原型实现

1. 在 NIO Server 的基础上，抽取了一个 `ChannelHandler`，用于处理连接和读取事件。
2. 以及分离了连接和读取事件所实现的位置，分别丢给 `Accept ChannelHandler` 和 `Read ChannelHandler`  来处理，以实现业务解耦。
3. 这就是 Reactor 的思路原型，其中还有很多地方可以继续抽象优化的~

```java
public class BaseReactorServer {

    interface ChannelHandler {
        void onRead(SocketChannel channel) throws Exception;
        void onAccept();
    }

    public static void start(int port) throws Exception {
        // non blocking
        final ServerSocketChannel serverChannel = ServerSocketChannel.open();
        serverChannel.configureBlocking(false);

        //bind & listen
        InetSocketAddress address = new InetSocketAddress(port);
        serverChannel.bind(address);

        final Selector selector = Selector.open();
        SelectionKey selectionKey = serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.err.println("注册事件: OP_ACCEPT");

        // Acceptor: 绑定OP_ACCEPT
        selectionKey.attach(new ChannelHandler() {
            public void onRead(SocketChannel channel) {

            }

            // ChannelHandler: 绑定OP_READ
            public void onAccept() {
                try {
                    SocketChannel socket = serverChannel.accept();
                    System.out.println("Accept !");
                    socket.configureBlocking(false);
                    SelectionKey sk = socket.register(selector, SelectionKey.OP_READ);

                    // 绑定
                    sk.attach(new ChannelHandler() {
                        public void onRead(SocketChannel socket) throws IOException {
                            final ByteBuffer buffer = ByteBuffer.allocate(256);
                            final int bytesRead = socket.read(buffer);

                            // Worker
                            if (bytesRead > 0) {
                                // 读
                                String readRes = new String(buffer.array()).trim();
                                System.err.println("read: " + readRes);

                                // 模拟业务执行过久
                                doBusiness(500, readRes);

                                // 写
                                buffer.flip();
                                socket.write(buffer);
                                buffer.clear();
                            } else if (bytesRead < 0) {
                                socket.close();
                                System.out.println("Client close");
                            }
                        }

                        public void onAccept() {

                        }

                        // 模拟业务执行过久
                        private void doBusiness(long time, String readRes) {
                            System.err.println("测试业务处理: time=" + time);
                            try {
                                Thread.sleep(time);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    });
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });

        while (true) {
            selector.select();
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            Iterator<SelectionKey> it = readyKeys.iterator();
            while (it.hasNext()) {
                SelectionKey key = it.next();
                ChannelHandler handler = (ChannelHandler) key.attachment();
                System.err.println(String.format("handler: %s", handler));

                // OP_ACCEPT
                if (key.isAcceptable()) {
                    handler.onAccept();
                }
                // OP_READ
                else if (key.isReadable()) {
                    handler.onRead((SocketChannel) key.channel());
                }

                // 从readyKeys中移除, 免得下次重新执行
                it.remove();
            }
        }

    }

    public static void main(String[] args) throws Exception {
        start(8084);
    }
}
```

###### Reactor 架构图实现

Reactor 又称之为响应器模式，常用于 NIO 网络通信框架，其单线程服务架构图如下，不同于传统 I/O 的串行调度方式，NIO 把整个服务请求分为五个阶段：

1. **read**：接收到请求，读取数据。
2. **decode**：解码数据。
3. **compute**：业务逻辑处理。
4. **encode**：编码返回数据。
5. **send**：返回数据。

![1646831937063](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646831937063.png)

1. `selector.select()` 是前面 NIO#API （多路复用模型），可以实现一个阻塞对象监听多路的连接请求。
2. `Reactor` 对象通过 `selector.select()` 监控客户端请求事件，收到事件后通过 `dispatch()` 进行分发。
3. 如果是建立连接请求事件，则由 `Acceptor` 通过 `serverSocketChannel.accept()` 处理连接请求，然后创建一个 `Handler` 对象，来处理连接完成后的后续业务处理。
4. 如果不是建立连接事件，则 `Reactor` 会分发调用连接对应的 `Handler` 来响应。
5. `Handler` 会完成 `read()` -> `doBusiness()` 业务处理 -> `send()` 一个完整的业务流程。

- **优点**：服务器使用一个线程，通过多路复用搞定所有 I/O 操作（包括连接、读、写等），编码简单，清晰明了，但如果客户端连接数量较多，将无法支撑。
- **缺点**：
  1. **性能问题**：只有一个线程，无法完全发挥多核 CPU 的性能，Handler 在处理某个连接上业务时，整个进程无法处理其它连接事件，容易导致性能瓶颈。
  2. **可靠性问题**：线程意外终止，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障。

```java
/**
 * 1、单Reactor单线程
 */
public class AaReactorServer implements Runnable {

    private final ServerSocketChannel serverSocketChannel;
    private final Selector selector;

    public AaReactorServer(int port) throws IOException {
        // non blocking
        this.serverSocketChannel = ServerSocketChannel.open();
        this.serverSocketChannel.configureBlocking(false);

        // listen & bind
        this.serverSocketChannel.socket().bind(new InetSocketAddress(port));

        // Acceptor: 绑定OP_ACCEPT
        this.selector = Selector.open();
        SelectionKey selectionKey = this.serverSocketChannel.register(this.selector, SelectionKey.OP_ACCEPT);
        selectionKey.attach(new Acceptor(this.selector, this.serverSocketChannel));
    }

    @Override
    public void run() {
        while(!Thread.interrupted()) {
            System.out.println("Waiting for new event on port: " + this.serverSocketChannel.socket().getLocalPort() + "...");
            try {
                // 如果没有事件发生, 则继续自旋
                if(this.selector.select() == 0) {
                    continue;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            // 有事件发生, 则取出所有发生的事件
            Set<SelectionKey> selectionKeys = this.selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                // 根据事件进行派发
                dispatch(iterator.next());

                // 派发完毕, 从selectionKeys中移除, 免得下次重新执行
                iterator.remove();
            }
        }
    }

    // 根据事件进行派发
    private void dispatch(SelectionKey key) {
        // 根据key绑定的对象, 开辟新线程
        Runnable runnable = (Runnable) key.attachment();
        if(runnable != null) {
            runnable.run();
        }
    }

    public static void main(String[] args) {
        try {
            AaReactorServer aaReactorServer = new AaReactorServer(8084);
            aaReactorServer.run();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

/**
 * 2、Acceptor
 */
public class Acceptor implements Runnable {

    private final ServerSocketChannel serverSocketChannel;
    private final Selector selector;

    public Acceptor(Selector selector, ServerSocketChannel serverSocketChannel) {
        this.serverSocketChannel = serverSocketChannel;
        this.selector = selector;
    }

    @Override
    public void run() {
        try {
            // OP_ACCEPT
            SocketChannel socketChannel = serverSocketChannel.accept();
            System.out.println(socketChannel.socket().getRemoteSocketAddress().toString() + " is connected...");

            // non blocking
            socketChannel.configureBlocking(false);

            // ChannelHandler: 绑定OP_READ
            SelectionKey selectionKey = socketChannel.register(this.selector, SelectionKey.OP_READ);
            selectionKey.attach(new ChannelHandler(selectionKey, socketChannel));

            // 唤醒一个阻塞的selector
            this.selector.wakeup();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

/**
 * 3、ChannelHandler
 */
public class ChannelHandler implements Runnable {

    private final SelectionKey selectionKey;
    private final SocketChannel socketChannel;
    int state;

    public ChannelHandler(SelectionKey selectionKey, SocketChannel socketChannel) {
        this.selectionKey = selectionKey;
        this.socketChannel = socketChannel;

        // 初始默认为read状态
        state = 0;
    }

    @Override
    public void run() {
        try {
            // 读取
            if(this.state == 0) {
                read();
            }
            // 返回
            else {
                send();
            }
        } catch (IOException e) {
            System.out.println("Waring! A Client has been closed...");
            closeChannel();
        }
    }

    private synchronized void read() throws IOException {
        // non-blocking下不可用Readers, 因为Readers不支持non-blocking
        byte[] bytes = new byte[1024];
        ByteBuffer byteBuffer = ByteBuffer.wrap(bytes);

        // 读取字符串
        int numReadBytes = this.socketChannel.read(byteBuffer);
        if(numReadBytes == -1) {
            System.out.println("Warning! A client has bean closed...");
            closeChannel();
            return;
        }

        // byte[] => String
        String str = new String(bytes);
        if(!"".equals(str)) {
            // 执行业务处理
            doBusiness(str);
            System.out.println(socketChannel.socket().getRemoteSocketAddress().toString() + ">" + str);

            // 读取 -> 返回
            this.state = 1;
            this.selectionKey.interestOps(SelectionKey.OP_WRITE);

            // 唤醒一个阻塞的selector
            selectionKey.selector().wakeup();
        }
    }

    private void doBusiness(String str) {
        System.err.println("执行业务处理...");
    }

    private void closeChannel() {
        selectionKey.cancel();
        try {
            socketChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void send() throws IOException {
        // 返回数据
        String str = "Your message has sent to " + socketChannel.socket().getLocalSocketAddress().toString() + "\r\n";
        ByteBuffer byteBuffer = ByteBuffer.wrap(str.getBytes());
        while (byteBuffer.hasRemaining()) {
            socketChannel.write(byteBuffer);
        }

        // 返回 -> 读取
        this.state = 0;
        this.selectionKey.interestOps(SelectionKey.OP_READ);

        // 唤醒一个阻塞的selector
        selectionKey.selector().wakeup();
    }
}
```

##### 2）单 Reactor 多线程模式

![1646889343446](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646889343446.png)

1. `Reactor` 对象通过 `selector.select()` 监控客户端请求事件，收到事件后，通过 `dispatch()` 进行分发。
2. 如果是建立连接请求，则由 `Acceptor` 通过 `serverSocketChannel.accept()` 处理连接请求，然后创建一个 `Handler` 对象，来处理完成连接后的各种事件。
3. 如果不是连接请求，则由 `Reactor` 对象分发调用连接对应的 `Handler` 来处理。
4. 此时，`Handler` 只负责读取和响应事件，不做具体的业务处理，通过 `read()` 读取数据后，会分发给后面的 `Worker` 线程池的某个线程进行处理业务。
5. `Worker` 线程池会分配独立线程完成真正的业务，并将结果返回给 `Handler`。
6. `Handler` 收到线程池处理完的结果后，通过 `send()` 将结果返回给 Client。

- **优点**：可以充分利用多核 CPU 的处理能力。
- **缺点**：
  1. 多线程在需要数据共享时，可能实现比较复杂。
  2. Reactor 单线程处理完所有的监听、连接、读、写事件，在高并发场景下，容易出现性能瓶颈。

```java
/**
 * 1、单Reactor多线程
 */
public class AbReactorServer implements Runnable {

    private final ServerSocketChannel serverSocketChannel;
    private final Selector selector;

    public AbReactorServer(int port) throws IOException {
        // non blocking
        this.serverSocketChannel = ServerSocketChannel.open();
        this.serverSocketChannel.configureBlocking(false);

        // listen & bind
        this.serverSocketChannel.socket().bind(new InetSocketAddress(port));

        // Acceptor: 绑定OP_ACCEPT
        this.selector = Selector.open();
        SelectionKey selectionKey = this.serverSocketChannel.register(this.selector, SelectionKey.OP_ACCEPT);
        selectionKey.attach(new Acceptor(this.selector, this.serverSocketChannel));
    }

    @Override
    public void run() {
        while(!Thread.interrupted()) {
            System.out.println("Waiting for new event on port: " + this.serverSocketChannel.socket().getLocalPort() + "...");
            try {
                // 如果没有事件发生, 则继续自旋
                if(this.selector.select() == 0) {
                    continue;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            // 有事件发生, 则取出所有发生的事件
            Set<SelectionKey> selectionKeys = this.selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                // 根据事件进行派发
                dispatch(iterator.next());

                // 派发完毕, 从selectionKeys中移除, 免得下次重新执行
                iterator.remove();
            }
        }
    }

    // 根据事件进行派发
    private void dispatch(SelectionKey key) {
        // 根据key绑定的对象, 开辟新线程
        Runnable runnable = (Runnable) key.attachment();
        if(runnable != null) {
            runnable.run();
        }
    }

    public static void main(String[] args) {
        try {
            AbReactorServer abReactorServer = new AbReactorServer(8084);
            abReactorServer.run();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

/**
 * 2、Acceptor
 */
public class Acceptor implements Runnable {

    private final ServerSocketChannel serverSocketChannel;
    private final Selector selector;

    public Acceptor(Selector selector, ServerSocketChannel serverSocketChannel) {
        this.serverSocketChannel = serverSocketChannel;
        this.selector = selector;
    }

    @Override
    public void run() {
        try {
            // OP_ACCEPT
            SocketChannel socketChannel = serverSocketChannel.accept();
            System.out.println(socketChannel.socket().getRemoteSocketAddress().toString() + " is connected...");

            // non blocking
            socketChannel.configureBlocking(false);

            // ChannelHandler: 绑定OP_READ
            SelectionKey selectionKey = socketChannel.register(this.selector, SelectionKey.OP_READ);
            selectionKey.attach(new ChannelHandler(selectionKey, socketChannel));

            // 唤醒一个阻塞的selector
            this.selector.wakeup();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

/**
 * 3、ChannelHandler
 */
public class ChannelHandler implements Runnable {

    private static final int DEFAULT_THREAD_COUNT = 4;
    private static final int MAX_THREAD_COUNT = 8;
    private static final ThreadPoolExecutor POOL = new ThreadPoolExecutor(
            DEFAULT_THREAD_COUNT,
            MAX_THREAD_COUNT,
            10,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(1024)
    );

    private final SelectionKey selectionKey;
    private final SocketChannel socketChannel;
    private HandlerState state;

    public ChannelHandler(SelectionKey selectionKey, SocketChannel socketChannel) {
        this.selectionKey = selectionKey;
        this.socketChannel = socketChannel;

        // 初始默认为read状态
        state = new ReadState();
    }

    @Override
    public void run() {
        try {
            state.handle(this, this.selectionKey, this.socketChannel, POOL);
        } catch (IOException e) {
            System.out.println("Waring! A Client has been closed...");
            closeChannel();
        }
    }

    public void closeChannel() {
        selectionKey.cancel();
        try {
            socketChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void setState(HandlerState state) {
        this.state = state;
    }
}

/**
 * 4、状态
 */
public interface HandlerState {

    /**
     * 状态处理事件
     *
     * @param handler
     * @param selectionKey
     * @param socketChannel
     * @param pool
     */
    void handle(ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) throws IOException;
}

/**
 * 5、读取状态
 */
public class ReadState implements HandlerState {

    private SelectionKey selectionKey;

    @Override
    public void handle(ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) throws IOException {
        this.selectionKey = selectionKey;

        // non-blocking下不可用Readers, 因为Readers不支持non-blocking
        byte[] bytes = new byte[1024];
        ByteBuffer byteBuffer = ByteBuffer.wrap(bytes);

        // 读取字符串
        int numReadBytes = socketChannel.read(byteBuffer);
        if(numReadBytes == -1) {
            System.out.println("Warning! A client has bean closed...");
            handler.closeChannel();
            return;
        }

        // byte[] => String
        String str = new String(bytes);
        if(!"".equals(str)) {
            System.out.println(socketChannel.socket().getRemoteSocketAddress().toString() + ">" + str);

            // 开始处理业务
            handler.setState(new WorkState(str, handler, selectionKey, socketChannel, pool));
        }
    }
}

/**
 * 6、工作状态
 */
public class WorkState implements HandlerState {

    private String readResult;
    private SelectionKey selectionKey;

    public WorkState(String readResult, ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) {
        this.readResult = readResult;
        this.selectionKey = selectionKey;

        try {
            this.handle(handler, selectionKey, socketChannel, pool);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void handle(ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) throws IOException {
        pool.execute(new WorkerThread(handler, this.readResult));
    }

    private synchronized void change2WriteState(ChannelHandler handler, String str) {
        // 读取 -> 返回
        handler.setState(new WriteState());
        this.selectionKey.interestOps(SelectionKey.OP_WRITE);

        // 唤醒一个阻塞的selector
        this.selectionKey.selector().wakeup();
    }

    class WorkerThread implements Runnable {

        private ChannelHandler handler;
        private String str;

        public WorkerThread(ChannelHandler handler, String str) {
            this.handler = handler;
            this.str = str;
        }

        @Override
        public void run() {
            // 执行业务处理
            doBusiness(str);
            change2WriteState(this.handler, this.str);
        }

        private void doBusiness(String str) {
            System.err.println("执行业务处理...");
        }
    }
}

/**
 * 7、返回状态
 */
public class WriteState implements HandlerState {

    @Override
    public void handle(ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) throws IOException {
        // 返回数据
        String str = "Your message has sent to " + socketChannel.socket().getLocalSocketAddress().toString() + "\r\n";
        ByteBuffer byteBuffer = ByteBuffer.wrap(str.getBytes());
        while (byteBuffer.hasRemaining()) {
            socketChannel.write(byteBuffer);
        }

        // 返回 -> 读取
        handler.setState(new ReadState());
        selectionKey.interestOps(SelectionKey.OP_READ);

        // 唤醒一个阻塞的selector
        selectionKey.selector().wakeup();
    }
}
```

##### 3）主从 Reactor 多线程模式

![1646889441630](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646889441630.png)

1. `Reactor` 主线程 `MainReactor` 对象，通过 `selector.select()` 监听连接事件，收到事件后，通过 `Acceptor` 处理连接事件。
2. 当 `Acceptor` 处理连接事件后，`MainReactor` 轮训式地将连接分配给 `SubReactor`，其中，`Reactor` 主线程可以对应多个 `Reactor` 子线程，即 `MainReactor` 可以关联多个 `SubReactor`。
3. `SubReactor` 将连接加入到连接队列进行监听，并创建 `Handler` 进行各种事件处理。
4. 当有新事件发生时，`SubReactor` 就会调用对应的 `Handler` 进行处理。
5. `Handler` 通过 `read()` 读取数据，分发给后面线程池中的 `Worker` 线程进行处理。
6. `Worker` 线程池会分配独立的 worker 线程进行业务处理，并返回结果。
7. `Handler` 收到处理结果后，再通过 `send()` 将结果返回给 Client。

- **优点**：
  1. 父线程与子线程的数据交互简单职责明确，父线程只需要接收新连接，子线程完成后续的业务处理。
  2. 父线程与子线程的数据交互简单，Reactor 主线程只需要把新连接传给子线程，子线程无需返回数据。
- **缺点**：编程复杂度较高。

=> 这种模型在许多项目中广泛使用，包括 Nginx 主从 Reactor 多进程模型，Memcached 主从多线程，Netty 主从多线程模型的支持。

```java
/**
 * 1、主从Reactor
 */
public class BbReactorServer implements Runnable {

    private final ServerSocketChannel serverSocketChannel;
    private final Selector selector;

    public BbReactorServer(int port) throws IOException {
        // non blocking
        this.serverSocketChannel = ServerSocketChannel.open();
        this.serverSocketChannel.configureBlocking(false);

        // listen & bind
        this.serverSocketChannel.socket().bind(new InetSocketAddress(port));

        // Acceptor: 绑定OP_ACCEPT, 主selector用于处理连接
        this.selector = Selector.open();
        SelectionKey selectionKey = this.serverSocketChannel.register(this.selector, SelectionKey.OP_ACCEPT);
        selectionKey.attach(new Acceptor(this.serverSocketChannel));
    }

    @Override
    public void run() {
        while(!Thread.interrupted()) {
            System.out.println("MainReactor waiting for new event on port: " + this.serverSocketChannel.socket().getLocalPort() + "...");
            try {
                // 如果没有事件发生, 则继续自旋
                if(this.selector.select() == 0) {
                    continue;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            // 有事件发生, 则取出所有发生的事件
            Set<SelectionKey> selectionKeys = this.selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                // 根据事件进行派发
                dispatch(iterator.next());

                // 派发完毕, 从selectionKeys中移除, 免得下次重新执行
                iterator.remove();
            }
        }
    }

    // 根据事件进行派发
    private void dispatch(SelectionKey key) {
        // 根据key绑定的对象, 开辟新线程
        Runnable runnable = (Runnable) key.attachment();
        if(runnable != null) {
            runnable.run();
        }
    }

    public static void main(String[] args) {
        try {
            BbReactorServer bbReactorServer = new BbReactorServer(8084);
            bbReactorServer.run();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

/**
 * 2、Acceptor
 */
public class Acceptor implements Runnable {

    private static final int CORES = Runtime.getRuntime().availableProcessors();
    private final ServerSocketChannel serverSocketChannel;
    private final Selector[] selectors = new Selector[CORES];
    private int selIndex = 0;
    private SubReactor[] subReactors = new SubReactor[CORES];
    private Thread[] threads = new Thread[CORES];

    public Acceptor(ServerSocketChannel serverSocketChannel) throws IOException {
        this.serverSocketChannel = serverSocketChannel;

        // 创建多个selector, 以及多个SubReactor线程
        for (int i = 0; i < CORES; i++) {
            this.selectors[i] = Selector.open();
            this.subReactors[i] = new SubReactor(this.selectors[i], serverSocketChannel, i);
            this.threads[i] = new Thread(this.subReactors[i]);
            this.threads[i].start();
        }
    }

    @Override
    public void run() {
        try {
            // OP_ACCEPT
            SocketChannel socketChannel = this.serverSocketChannel.accept();
            System.out.println(socketChannel.socket().getRemoteSocketAddress().toString() + " is connected...");

            // non blocking
            socketChannel.configureBlocking(false);

            // 暂停SubReactor线程, 停止轮训
            SubReactor subReactor = this.subReactors[this.selIndex];
            Selector selector = this.selectors[this.selIndex];
            Thread thread = this.threads[this.selIndex];
            subReactor.setRestart(true);

            // 唤醒一个阻塞的selector
            selector.wakeup();

            // ChannelHandler: 绑定OP_READ, 从selector用于处理读取、返回
            SelectionKey selectionKey = socketChannel.register(selector, SelectionKey.OP_READ);
            selectionKey.attach(new ChannelHandler(selectionKey, socketChannel));

            // 重启SubReactor线程, 只有为false才会继续轮训
            subReactor.setRestart(false);
            LockSupport.unpark(thread);

            // 轮训重置selIndex
            if(++this.selIndex == this.selectors.length) {
                this.selIndex = 0;
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

/**
 * 3、子Reactor
 */
public class SubReactor implements Runnable {

    private final ServerSocketChannel serverSocketChannel;
    private final Selector selector;
    private volatile boolean restart = false;
    private int num;

    public SubReactor(Selector selector, ServerSocketChannel serverSocketChannel, int num) {
        this.serverSocketChannel = serverSocketChannel;
        this.selector = selector;
        this.num = num;
    }

    @Override
    public void run() {
        while(!Thread.interrupted()) {
            if(this.restart) {
                LockSupport.park();
            }

            System.out.println("SubReactor"+ this.num + " waiting for new event on port: " + this.serverSocketChannel.socket().getLocalPort() + "...");
            try {
                // 如果没有事件发生, 则继续自旋
                if(this.selector.select() == 0) {
                    continue;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            // 有事件发生, 则取出所有发生的事件
            Set<SelectionKey> selectionKeys = this.selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                // 根据事件进行派发
                dispatch(iterator.next());

                // 派发完毕, 从selectionKeys中移除, 免得下次重新执行
                iterator.remove();
            }
        }
    }

    // 根据事件进行派发
    private void dispatch(SelectionKey key) {
        // 根据key绑定的对象, 开辟新线程
        Runnable runnable = (Runnable) key.attachment();
        if(runnable != null) {
            runnable.run();
        }
    }

    // 重启SubReactor线程, 只有为false才会继续轮训
    public void setRestart(boolean restart) {
        this.restart = restart;
    }
}

/**
 * 4、ChannelHandler
 */
public class ChannelHandler implements Runnable {

    private static final int DEFAULT_THREAD_COUNT = 4;
    private static final int MAX_THREAD_COUNT = 8;
    private static final ThreadPoolExecutor POOL = new ThreadPoolExecutor(
            DEFAULT_THREAD_COUNT,
            MAX_THREAD_COUNT,
            10,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(1024)
    );

    private final SelectionKey selectionKey;
    private final SocketChannel socketChannel;
    private HandlerState state;

    public ChannelHandler(SelectionKey selectionKey, SocketChannel socketChannel) {
        this.selectionKey = selectionKey;
        this.socketChannel = socketChannel;

        // 初始默认为read状态
        state = new ReadState();
    }

    @Override
    public void run() {
        try {
            state.handle(this, this.selectionKey, this.socketChannel, POOL);
        } catch (IOException e) {
            System.out.println("Waring! A Client has been closed...");
            closeChannel();
        }
    }

    public void closeChannel() {
        selectionKey.cancel();
        try {
            socketChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void setState(HandlerState state) {
        this.state = state;
    }
}

/**
 * 5、状态
 */
public interface HandlerState {

    /**
     * 状态处理事件
     *
     * @param handler
     * @param selectionKey
     * @param socketChannel
     * @param pool
     */
    void handle(ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) throws IOException;
}

/**
 * 6、读取状态
 */
public class ReadState implements HandlerState {

    private SelectionKey selectionKey;

    @Override
    public void handle(ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) throws IOException {
        this.selectionKey = selectionKey;

        // non-blocking下不可用Readers, 因为Readers不支持non-blocking
        byte[] bytes = new byte[1024];
        ByteBuffer byteBuffer = ByteBuffer.wrap(bytes);

        // 读取字符串
        int numReadBytes = socketChannel.read(byteBuffer);
        if(numReadBytes == -1) {
            System.out.println("Warning! A client has bean closed...");
            handler.closeChannel();
            return;
        }

        // byte[] => String
        String str = new String(bytes);
        if(!"".equals(str)) {
            System.out.println(socketChannel.socket().getRemoteSocketAddress().toString() + ">" + str);

            // 开始处理业务
            handler.setState(new WorkState(str, handler, selectionKey, socketChannel, pool));
        }
    }
}

/**
 * 7、工作状态
 */
public class WorkState implements HandlerState {

    private String readResult;
    private SelectionKey selectionKey;

    public WorkState(String readResult, ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) {
        this.readResult = readResult;
        this.selectionKey = selectionKey;

        try {
            this.handle(handler, selectionKey, socketChannel, pool);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void handle(ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) throws IOException {
        pool.execute(new WorkerThread(handler, this.readResult));
    }

    private synchronized void change2WriteState(ChannelHandler handler, String str) {
        // 读取 -> 返回
        handler.setState(new WriteState());
        this.selectionKey.interestOps(SelectionKey.OP_WRITE);

        // 唤醒一个阻塞的selector
        this.selectionKey.selector().wakeup();
    }

    // 工作线程用于处理业务逻辑
    class WorkerThread implements Runnable {

        private ChannelHandler handler;
        private String str;

        public WorkerThread(ChannelHandler handler, String str) {
            this.handler = handler;
            this.str = str;
        }

        @Override
        public void run() {
            // 执行业务处理
            doBusiness(str);
            change2WriteState(this.handler, this.str);
        }

        private void doBusiness(String str) {
            System.err.println("执行业务处理...");
        }
    }
}

/**
 * 8、返回状态
 */
public class WriteState implements HandlerState {

    @Override
    public void handle(ChannelHandler handler, SelectionKey selectionKey, SocketChannel socketChannel, ThreadPoolExecutor pool) throws IOException {
        // 返回数据
        String str = "Your message has sent to " + socketChannel.socket().getLocalSocketAddress().toString() + "\r\n";
        ByteBuffer byteBuffer = ByteBuffer.wrap(str.getBytes());
        while (byteBuffer.hasRemaining()) {
            socketChannel.write(byteBuffer);
        }

        // 返回 -> 读取
        handler.setState(new ReadState());
        selectionKey.interestOps(SelectionKey.OP_READ);

        // 唤醒一个阻塞的selector
        selectionKey.selector().wakeup();
    }
}
```

##### Reactor 模式总结

| Reactor 线程模型    | 比喻                                           |
| ------------------- | ---------------------------------------------- |
| 单 Reactor 单线程   | 前台接待员和服务员都是同一个人，全程为顾客服务 |
| 单 Reactor 多线程   | 1 个前台接待员，多个服务员，接待员只负责接待   |
| 主从 Reactor 多线程 | 多个前台接待员，多个服务生                     |

**Reactor 优点**：

1. 响应快，不必被单个同步时间所阻塞，虽然 Reactor 本身依然是同步的。
2. 可以最大程度地避免复杂了多线程及同步问题，并且避免了多线程 / 进程切换的开销。
3. 扩展性好，可以方便的通过增加 Reactor 实例个数，来充分利用 CPU 资源。
4. 复用性好，Reactor 模型本身与具体事件处理逻辑无关，具有很高的复用性。

#### Proactor 模式

Proactor 模式 read 操作过程为：

1. 应用程序初始化一个异步读取操作，然后注册相应的事件处理器，此时事件处理器不关注 `读取就绪` 事件，而是关注 `读完成` 事件，这是区别于 Reactor 的关键。
2. 事件分离器等待 `读完成` 事件。
3. 在事件分离器等待 `读完成` 时，操作系统调用内核线程完成读取操作，并将读取的内容放入用户传递过来的缓存区中，这也是区别于 Reactor的一点，Proactor 中，应用程序**需传递缓存区**。
4. 事件分离器捕获到 `读完成` 事件后，激活应用程序注册的事件处理器，事件处理器直接从缓存区读取数据，而不需要进行实际的读取操作。
5. 而 Proactor 中的 write 操作和 read 操作类似，即感兴趣的事件是 `写完成` 事件。

![1646831843451](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646831843451.png)

=> 从上面可以看出，Reactor 和 Proactor 模式的主要区别就是，真正的读取和写入操作是由谁来完成的，Reactor 中需要应用程序自己读取或者写入数据，而 Proactor 模式中，应用程序不需要进行实际的读写过程，只需要从缓存区读取或者写入即可，操作系统会写入缓存区或者从缓存区读取并写入到真正的 I/O 设备中。

### 2.7. 什么是 Netty？

- **概念**：Netty 是一个 基于 JAVA NIO 实现的高性能、异步事件驱动的 NIO 框架。
- **原理**：它提供了对TCP、UDP 和文件传输的支持，作为一个异步实现的 NIO 框架，Netty 的所有 I/O 操作都是异步非阻塞的，通过 `Future-Listener` 机制，用户可以方便地主动获取，或者通过通知机制获得 I/O 操作结果。 
- **高性能原理**：
  1. **I/O 多路复用**：见 I/O 多路复用模型。
  2. **Reactor 线程模型**：见主从 Reactor 模型。
  3. **零拷贝机制**：
     1. Netty 接收和发送的 ByteBuffer，底层采用`DirectBuffers` 使用堆外内存进行 Socket 读写，不需要进行字节缓冲区的二次拷贝。
     2. 如果使用传统的堆内存 `HEAP BUFFERS` 进行Socket读写，JVM 会将堆内存 Buffer 拷贝一份到直接内存中，然后才写入 Socket 中，此时相比于堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝。
     3. Netty 提供了组合 Buffer 对象，可以聚合多个 ByteBuffer 对象，用户可以像操作一个 Buffer 那样
        方便，对组合 Buffer 进行操作，避免了传统通过内存拷贝的方式，将几个小 Buffer 合并成一个大的
        Buffer。 
     4. Netty 文件传输采用了 `transferTo()` 方法，它可以直接将文件缓冲区的数据发送到目标 Channel，
        避免了传统通过循环 `write()`方式导致的内存拷贝问题。
  4. **高性能的序列化框架**：Netty 默认提供了对 Google Protobuf 的支持，通过扩展 Netty 的编解码接口，用户可以自定义实现其它的高性能序列化框架，比如 Thrift 的压缩二进制编解码框架等。 

### 2.8. Netty 快速入门？

Netty 实现通信的步骤：（客户端与服务端基本一致）

1. 先创建 2 个 NIO 线程组，一个专门用于网络事件的处理（接受客户端的连接），另一个则进行网络通信读写。
2. 然后，创建一个 `ServerBootStrap` 对象，配置 Netty 的一些列参数，比如接受传出数据的缓存大小等等。
3. 接着，创建一个实际处理数据的 `ChannelInitializer` 类，进行初始化的准备工作，比如设置接受传出数据的字符集、格式、实际处理数据的 `ChannelHandler` 等。
4. 最后，绑定接口，执行同步阻塞方法，等待服务端启动即可。

#### 1、NettyServer

```java
/**
 * 1、NettyServer
 */
public class NettyServer {

    public static void main(String[] args) throws InterruptedException {
        // 1. 创建两个线程组: 一个是用于处理服务端接收客户端连接, 一个用于进行网络通信(即网络读写)
        NioEventLoopGroup parentGroup = new NioEventLoopGroup();
        NioEventLoopGroup childGroup = new NioEventLoopGroup();

        // 2. 创建辅助配置工具类, 用于服务器通道的一系列配置
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap
                // 2.1. 绑定两个线程组, 父子线程组
                .group(parentGroup, childGroup)
                // 2.2. 指定NIO模式, 因为是Server端, 所以要绑定NioServerSocketChannel
                .channel(NioServerSocketChannel.class)
                // 2.3. 指定连接超时时间
                .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 3000)
                // 2.4. TCP不允许延迟, 通信不延迟
                .option(ChannelOption.TCP_NODELAY, true)
                // 2.5. 设置TCP缓冲区大小 eg => 32M = sync队列 + accept队列
                .option(ChannelOption.SO_BACKLOG, 32 * 1024)
                // 2.6. 设置接收缓冲区大小
                .option(ChannelOption.SO_RCVBUF, 32 * 1024)
                // 2.6.1. 设置接收缓冲区自动扩容, 已弃用
//                .option(ChannelOption.RCVBUF_ALLOCATOR, AdaptiveRecvByteBufAllocator.DEFAULT)
                // 2.6.2. 设置发送缓冲区使用对象池, 重用缓冲区
//                .option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
                // 2.7. 进行初始化ChannelInitializer , 用于构建双向链表 "pipeline" 添加业务handler处理
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        // 2.7.1. 配置自定义具体业务接收和处理的方法
                        ch.pipeline().addLast(new NettyServerHandler());
                    }
                });

        // 3. 使用辅助配置工具类绑定要监听的端口 => 同步阻塞
        ChannelFuture channelFuture = serverBootstrap.bind(8765).sync();

        // 4. 同步阻塞等待通道关闭 => 如果两边都不关闭, 则客户端和服务端的通道都会一直开着
        channelFuture.channel().closeFuture().sync();

        // 5. 释放资源
        parentGroup.shutdownGracefully();
        childGroup.shutdownGracefully();
    }
}
```

#### 2、NettyServerHandler

```java
/**
 * 2、NettyServerHandler
 */
public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     * 通道激活方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel active... ");
    }

    /**
     * 通道关闭方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel inactive... ");
    }

    /**
     * 读写数据核心方法 => 收到客户端连接则进行处理
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 1. 获取TCP包缓冲区数据
        ByteBuf byteBuf = (ByteBuf) msg;

        // 2. 根据缓冲数据大小构造字节数组
        byte[] bytes = new byte[byteBuf.readableBytes()];

        // 3. 读取到缓冲区数据到字节数组中
        byteBuf.readBytes(bytes);

        // 4. 使用UTF-8编码解码字节数组成字符串
        String body = new String(bytes, "utf-8");
        System.err.println("Netty server: " + body);

        // 5. 构造响应体给客户端 => 测试与客户端的交互
        String response = "Netty server ack: " + body;
        ctx.writeAndFlush(Unpooled.copiedBuffer(response.getBytes()));
    }

    /**
     * 读写数据完毕方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel read complete... ");
    }

    /**
     * 捕获异常方法
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        super.exceptionCaught(ctx, cause);
    }
}
```

#### 3、NettyClient

```java
/**
 * 3、NettyClient
 */
public class NettyClient {

    public static void main(String[] args) throws InterruptedException {
        // 1. 创建一个线程组: 只需要一个线程组用于实际业务的处理(网络通信的读写)
        NioEventLoopGroup workGroup = new NioEventLoopGroup();

        // 2. 创建辅助配置工具类, 进行配置响应的参数 => 用于构造Client
        Bootstrap bootstrap = new Bootstrap();
        bootstrap
                // 2.1. 绑定线程组
                .group(workGroup)
                // 2.2. 指定NIO模式, 因为是Client端, 所以要绑定NioSocketChannel
                .channel(NioSocketChannel.class)
                // 2.3. 指定连接超时时间
                .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 3000)
                // 2.4. TCP不允许延迟, 通信不延迟
                .option(ChannelOption.TCP_NODELAY, true)
                // 2.5. 设置TCP缓冲区大小 eg => 32M = sync队列 + accept队列
                .option(ChannelOption.SO_BACKLOG, 32 * 1024)
                // 2.6. 设置接收缓冲区大小
                .option(ChannelOption.SO_RCVBUF, 32 * 1024)
                // 2.7. 进行初始化ChannelInitializer , 用于构建双向链表 "pipeline" 添加业务handler处理
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        // 2.7.1. 配置自定义具体业务接收和处理的方法
                        ch.pipeline().addLast(new NettyClientHandler());
                    }
                });

        // 3. 使用辅助配置工具类绑定要监听的端口, 并启动服务 => 同步阻塞
        ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 8765).syncUninterruptibly();

        // 4. 打破连接的同步阻塞, 则发送一条数据到服务器端
        channelFuture.channel().writeAndFlush(Unpooled.copiedBuffer("Hello netty!".getBytes()));

        // 5. 睡眠10秒钟后再发送一条数据到服务端
        Thread.sleep(10000);
        channelFuture.channel().writeAndFlush(Unpooled.copiedBuffer("Hello netty again!".getBytes()));

        // 6. 同步阻塞5s, 再关闭监听并释放资源
        cf.channel().closeFuture().await(5, TimeUnit.SECONDS);
        
        // 7. 释放资源
        workGroup.shutdownGracefully();
    }
}

```

#### 4、ClientHandler

```java
/**
 * 4、ClientHandler
 */
public class NettyClientHandler extends ChannelInboundHandlerAdapter {

    /**
     * 通道激活方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty client channel active... ");
    }

    /**
     * 通道关闭方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty client channel inactive... ");
    }

    /**
     * 读写数据核心方法 => 用于处理服务端回写的数据
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 固定模式的 try .. finally
        // 在try代码片段处理逻辑, finally进行释放缓存资源, 也就是 Object msg (buffer)
        try {
            // 1. 获取TCP包缓冲区数据
            ByteBuf byteBuf = (ByteBuf) msg;

            // 2. 根据缓冲数据大小构造字节数组
            byte[] bytes = new byte[byteBuf.readableBytes()];

            // 3. 读取到缓冲区数据到字节数组中
            byteBuf.readBytes(bytes);

            // 4. 使用UTF-8编码解码字节数组成字符串
            String body = new String(bytes, "utf-8");

            // 5. 构造响应体给客户端 => 测试与客户端的交互
            System.err.println("Netty client receive ack: " + body);
        } finally {
            // 6. 释放资源
            ReferenceCountUtil.release(msg);
        }
    }

    /**
     * 读写数据完毕方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel read complete... ");
    }

    /**
     * 捕获异常方法
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        // 客户端读写异常, 则关闭连接
        ctx.close();
    }
}
```

### 2.9. RPC、HTTP、MQ、Netty？

| 技术  | 应用场景                                          |
| ----- | ------------------------------------------------- |
| RPC   | 系统间即时访问、同步服务调用                      |
| HTTP  | 外部接口 API 提供、非高并发场景、非大数据报文传输 |
| MQ    | 微服务之间解耦、流量削峰                          |
| Netty | 底层基础通信、数据传输、数据同步                  |

### 3.0 Netty 核心组件？

#### 1）Channel

Channel，通道，连接是单向的，**通道双向的**，是Java NIO 的一个基本构造，可以看作是传入或传出数据的载体，它可以被打开或关闭，连接或者断开连接。以下是常用的 Channel 有 `EmbeddedChannel` 、`LocalServerChannel` 、`NioDatagramChannel` 、`NioSctpChannel` 、`NioSocketChannel`。

#### 2）Future 

1. Netty 中所有的 I/O 操作都是异步的，一个操作可能不会立即返回，可以在之后的某个时间点确定其结果的方法。
2. 和 `Handler` 回调 是相互补充的机制，提供了另一种在操作完成时通知应用程序的方式，可以看作是一个异步操作结果的占位符，将在未来的某个时刻完成，并提供对其结果的访问。
3. Netty 提供了 `ChannelFuture`，用于在执行异步操作的时候使用，每个 Netty 的 I/O 操作都会返回一个`ChannelFuture`。
4. `ChannelFuture` 能够注册一个或者多个 `ChannelFutureListener` 实例，监听器的回调方法`operationComplete()` 将会在对应的操作完成时被调用。

#### 3）ChannelHandler

1. Netty 的主要组件是 ChannelHandler，它负责所有处理出入站数据的逻辑处理。
2. Netty 使用不同的事件，来通知操作状态的改变，每个事件都可以被分发给 ChannelHandler 类中某个用户自定义实现的方法中。
3. Netty 还提供了大量预定义的、可以开箱即用的 ChannelHandler 实现，包括用于各种协议的ChannelHandler。

#### 4）ChannelPipeline

1. ChannelPipeline 提供了 ChannelHandler 链的容器，并定义了用于在该链上传播入站和出站事件流的 API，在应用程序的初始化、或者引导阶段被安装。
2. 这可以使事件流经 ChannelPipeline 时，让 ChannelHandler 进行相关工作，然后将数据传递给链中的下一个 ChannelHandler。

![1646917753469](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646917753469.png)

#### 5）EventLoop

1. EventLoop 定义了 Netty 的核心抽象，用来处理连接的生命周期中所发生的事件，在内部，将会为每个Channel 分配一个 EventLoop 。
2. EventLoop 本身只由一个线程驱动，其处理了一个 Channel 的所有 I/O 事件，并且在该 EventLoop 的整个生命周期内都不会改变。
3. EventLoop 的管理是通过 EventLoopGroup 来实现的。

![1646918023384](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646918023384.png)

#### 6）BootStarp、ServerBootstrap 

1. BootStarp 和 ServerBootstrap 被称为引导类，指对应用程序进行配置，并使他运行起来的过程。

2. BootStrap 是客户端的引导类，Bootstrap 在调用 `bind()`（连接UDP）和 `connect()`（连接TCP）方法时，会新创建一个 Channel，仅创建一个单独的、没有父 Channel 的 Channel 来实现所有的网络交换，只需要一个 EventLoopGroup。

   ![1646918238983](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646918238983.png)

3. ServerBootstrap 是服务端的引导类，ServerBootstarp 在调用 `bind()` 方法时，会创建一个 ServerChannel 来接受来自客户端的连接，并且该 ServerChannel 管理了多个子 Channel，用于同客户端之间的通信，通常需要两个 EventLoopGroup，一个用来接收客户端连接，一个用来处理 I/O 事件。

   ![1646918257373](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1646918257373.png)

#### 7）ByteBuffer

1. Netty 接收和发送的 ByteBuffer，底层采用`DirectBuffers` 使用堆外内存进行 Socket 读写，不需要进行字节缓冲区的二次拷贝。
2. 如果使用传统的堆内存 `HEAP BUFFERS` 进行Socket读写，JVM 会将堆内存 Buffer 拷贝一份到直接内存中，然后才写入 Socket 中，此时相比于堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝。
3. Netty 提供了组合 Buffer 对象，可以聚合多个 ByteBuffer 对象，用户可以像操作一个 Buffer 那样
   方便，对组合 Buffer 进行操作，避免了传统通过内存拷贝的方式，将几个小 Buffer 合并成一个大的
   Buffer。 
4. Netty 文件传输采用了 `transferTo()` 方法，它可以直接将文件缓冲区的数据发送到目标 Channel，
   避免了传统通过循环 `write()`方式导致的内存拷贝问题。
5.  

### 3.1. Netty 拆包粘包问题？

#### TCP 拆包粘包机制

1. 由于 TCP 是基于流的协议，本身没有界限，没有任何分界线，它不会理解上层数据包的含义。
2. 所以，一个上层的数据包可能会被拆成多个包，分批发送，这就是**拆包问题**。
3. 同理，多个上层小的数据包也能会被合成一个大包，一起发送，这就是**粘包问题**。

#### 问题产生原因

1. 应用程序写入的字节大小，大于套接字发送缓冲区的大小。
2. 进行 MSS 大小的 TCP 分段、以太网帧的 payload 大于 MTU 进行 IP 分片等。

#### 业务主流解决方案

1. **消息定长**：比如，每个报文大小固定为 200 个字节，如果不够，则用空格补位。
2. **在包尾添加特殊字符进行分割**：比如加回车符等。
3. **消息分为消息头 Header 和消息体 Body**：在消息头中包含表示消息总长度的字段，然后根据长度去读取消息体，再进行业务处理。
   - 用得最多，比如自定义协议栈。

##### 1）Netty 消息定长解决方案 | FixedLengthFrameDecoder

###### 1、NettyServer

```java
/**
 * 测试Netty: 服务端, 测试消息定长方式解决TCP拆包/粘包问题
 */
public class Pkg1NettyServer {

    public static void main(String[] args) throws InterruptedException {
        // 1. 创建两个线程组: 一个是用于处理服务端接收客户端连接, 一个用于进行网络通信(即网络读写)
        NioEventLoopGroup parentGroup = new NioEventLoopGroup();
        NioEventLoopGroup childGroup = new NioEventLoopGroup();

        // 2. 创建辅助配置工具类, 用于服务器通道的一系列配置
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap
                // 2.1. 绑定两个线程组, 父子线程组
                .group(parentGroup, childGroup)
                // 2.2. 指定NIO模式, 因为是Server端, 所以要绑定NioServerSocketChannel
                .channel(NioServerSocketChannel.class)
                // 2.3. 设置TCP缓冲区大小 eg => 32M = sync队列 + accept队列
                .option(ChannelOption.SO_BACKLOG, 32 * 1024)
                // 2.4. 设置接收缓冲区大小
                .option(ChannelOption.SO_RCVBUF, 32 * 1024)
                // 2.5. 进行初始化ChannelInitializer , 用于构建双向链表 "pipeline" 添加业务handler处理
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        // 2.5.1. 测试消息定长, 空格补位解决TCP拆包/粘包问题 => 定长5个字符
                        ch.pipeline().addLast(new FixedLengthFrameDecoder(5));

                        // 2.3.2. Netty String解码器
                        ch.pipeline().addLast(new StringDecoder());

                        // 2.5.3. 配置自定义具体业务接收和处理的方法
                        ch.pipeline().addLast(new Pkg1NettyServerHandler());
                    }
                });

        // 3. 使用辅助配置工具类绑定要监听的端口 => 同步阻塞
        ChannelFuture channelFuture = serverBootstrap.bind(8765).sync();

        // 4. 同步阻塞等待通道关闭 => 如果两边都不关闭, 则客户端和服务端的通道都会一直开着
        channelFuture.channel().closeFuture().sync();

        // 5. 释放资源
        parentGroup.shutdownGracefully();
        childGroup.shutdownGracefully();
    }
}
```

###### 2、NettyServerHandler

```java
/**
 * 测试Netty: 服务端业务处理器, 测试消息定长方式解决TCP拆包/粘包问题
 */
public class Pkg1NettyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     * 通道激活方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel active... ");
    }

    /**
     * 通道关闭方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel inactive... ");
    }

    /**
     * 读写数据核心方法 => 收到客户端连接则进行处理
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 1. 使用了Netty String解码器后, 可以直接转换成String类型
        String body = (String) msg;
        System.err.println("Netty server: " + body);

        // 2. 构造响应体给客户端 => 测试与客户端的交互
        ctx.writeAndFlush(Unpooled.copiedBuffer(body.getBytes()));
    }

    /**
     * 读写数据完毕方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel read complete... ");
    }

    /**
     * 捕获异常方法
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        super.exceptionCaught(ctx, cause);
    }
}
```

###### 3、NettyClient

```java
/**
 * 测试Netty: 客户端, 测试消息定长方式解决TCP拆包/粘包问题
 */
public class Pkg1NettyClient {

    public static void main(String[] args) throws InterruptedException {
        // 1. 创建一个线程组: 只需要一个线程组用于实际业务的处理(网络通信的读写)
        NioEventLoopGroup workGroup = new NioEventLoopGroup();

        // 2. 创建辅助配置工具类, 进行配置响应的参数 => 用于构造Client
        Bootstrap bootstrap = new Bootstrap();
        bootstrap
                // 2.1. 绑定线程组
                .group(workGroup)
                // 2.2. 指定NIO模式, 因为是Client端, 所以要绑定NioSocketChannel
                .channel(NioSocketChannel.class)
                // 2.3. 进行初始化ChannelInitializer , 用于构建双向链表 "pipeline" 添加业务handler处理
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        // 2.3.1. 测试消息定长, 空格补位解决TCP拆包/粘包问题 => 定长5个字符
                        ch.pipeline().addLast(new FixedLengthFrameDecoder(5));

                        // 2.3.2. Netty String解码器
                        ch.pipeline().addLast(new StringDecoder());

                        // 2.3.2. 配置自定义具体业务接收和处理的方法
                        ch.pipeline().addLast(new Pkg1NettyClientHandler());
                    }
                });

        // 3. 使用辅助配置工具类绑定要监听的端口, 并启动服务 => 同步阻塞
        ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 8765).syncUninterruptibly();

        // 4. 打破连接的同步阻塞, 则发送一条数据到服务器端 => 5个a, 5个b
        channelFuture.channel().writeAndFlush(Unpooled.copiedBuffer("aaaaabbbbb".getBytes()));

        // 5. 睡眠10秒钟后再发送一条数据到服务端、
        System.err.println("睡10s...");
        Thread.sleep(10000);
//        channelFuture.channel().writeAndFlush(Unpooled.copiedBuffer("ccccccc   ".getBytes()));// => 7个c + 3个空格 => 后面2个c和3个空格可以发送
        channelFuture.channel().writeAndFlush(Unpooled.copiedBuffer("ccccccc".getBytes()));// => 7个c + 没有空格补位 => 后面2个c不可以发送

        // 5.1. 再睡眠10秒钟后再发送一条数据到服务端
        System.err.println("再睡10s...");
        Thread.sleep(10000);
        channelFuture.channel().writeAndFlush(Unpooled.copiedBuffer("ddd".getBytes()));// => 3个d, 凑够5个字符 => 之前残留的2个c在缓冲区中, 凑了3个d凑够5个字符后, 可以一起发送出去

        // 6. 阻塞的方式等待通道关闭 => 如果两边都不关闭, 则客户端和服务端的通道都会一直开着
        channelFuture.channel().closeFuture().sync();

        // 7. 释放资源
        workGroup.shutdownGracefully();
    }
}
```

###### 4、ClientHandler

```java
/**
 * 测试Netty: 客户端业务处理器, 测试消息定长方式解决TCP拆包/粘包问题
 */
public class Pkg1NettyClientHandler extends ChannelInboundHandlerAdapter {

    /**
     * 通道激活方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty client channel active... ");
    }

    /**
     * 通道关闭方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty client channel inactive... ");
    }

    /**
     * 读写数据核心方法 => 用于处理服务端回写的数据
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 固定模式的 try .. finally
        // 在try代码片段处理逻辑, finally进行释放缓存资源, 也就是 Object msg (buffer)
        try {
            // 1. 使用了Netty String解码器后, 可以直接转换成String类型
            String response = (String) msg;

            // 5. 构造响应体给客户端 => 测试与客户端的交互
            System.err.println("Netty client: " + response);
        } finally {
            // 6. 释放资源
            ReferenceCountUtil.release(msg);
        }
    }

    /**
     * 读写数据完毕方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel read complete... ");
    }

    /**
     * 捕获异常方法
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        // 客户端读写异常, 则关闭连接
        ctx.close();
    }
}
```

##### 2）Netty 特殊字符解决方案 | DelimiterBasedFrameDecoder

###### 1、NettyServer

```java
/**
 * 测试Netty: 服务端, 测试特殊字符方式解决TCP拆包/粘包问题
 */
public class Pkg2NettyServer {

    public static void main(String[] args) throws InterruptedException {
        // 1. 创建两个线程组: 一个是用于处理服务端接收客户端连接, 一个用于进行网络通信(即网络读写)
        NioEventLoopGroup parentGroup = new NioEventLoopGroup();
        NioEventLoopGroup childGroup = new NioEventLoopGroup();

        // 2. 创建辅助配置工具类, 用于服务器通道的一系列配置
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap
                // 2.1. 绑定两个线程组, 父子线程组
                .group(parentGroup, childGroup)
                // 2.2. 指定NIO模式, 因为是Server端, 所以要绑定NioServerSocketChannel
                .channel(NioServerSocketChannel.class)
                // 2.3. 设置TCP缓冲区大小 eg => 32M = sync队列 + accept队列
                .option(ChannelOption.SO_BACKLOG, 32 * 1024)
                // 2.4. 设置接收缓冲区大小
                .option(ChannelOption.SO_RCVBUF, 32 * 1024)
                // 2.5. 进行初始化ChannelInitializer , 用于构建双向链表 "pipeline" 添加业务handler处理
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        // 2.3.1. 测试特殊字符方式解决TCP拆包/粘包问题, 以"$_"结尾作为分割符(只要收到$_,就认为到这里就是一个数据包), 最大帧长1024bit
                        ByteBuf delimiterBuf = Unpooled.copiedBuffer("$_".getBytes());
                        ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, delimiterBuf));

                        // 2.3.2. Netty String解码器
                        ch.pipeline().addLast(new StringDecoder());

                        // 2.5.3. 配置自定义具体业务接收和处理的方法
                        ch.pipeline().addLast(new Pkg2NettyServerHandler());
                    }
                });

        // 3. 使用辅助配置工具类绑定要监听的端口 => 同步阻塞
        ChannelFuture channelFuture = serverBootstrap.bind(8765).sync();

        // 4. 同步阻塞等待通道关闭 => 如果两边都不关闭, 则客户端和服务端的通道都会一直开着
        channelFuture.channel().closeFuture().sync();

        // 5. 释放资源
        parentGroup.shutdownGracefully();
        childGroup.shutdownGracefully();
    }
}
```

###### 2、NettyServerHandler

```java
/**
 * 测试Netty: 服务端业务处理器, 测试特殊字符方式解决TCP拆包/粘包问题
 */
public class Pkg2NettyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     * 通道激活方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel active... ");
    }

    /**
     * 通道关闭方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel inactive... ");
    }

    /**
     * 读写数据核心方法 => 收到客户端连接则进行处理
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 1. 使用了Netty String解码器后, 可以直接转换成String类型
        String body = (String) msg;
        System.err.println("Netty server: " + body);

        // 2. 构造响应体给客户端 => 测试与客户端的交互
        ctx.writeAndFlush(Unpooled.copiedBuffer(("服务器响应: " + body + "$_").getBytes()));
    }

    /**
     * 读写数据完毕方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel read complete... ");
    }

    /**
     * 捕获异常方法
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        super.exceptionCaught(ctx, cause);
    }
}
```

###### 3、NettyClient

```java
/**
 * 测试Netty: 客户端, 测试特殊字符方式解决TCP拆包/粘包问题
 */
public class Pkg2NettyClient {

    public static void main(String[] args) throws InterruptedException {
        // 1. 创建一个线程组: 只需要一个线程组用于实际业务的处理(网络通信的读写)
        NioEventLoopGroup workGroup = new NioEventLoopGroup();

        // 2. 创建辅助配置工具类, 进行配置响应的参数 => 用于构造Client
        Bootstrap bootstrap = new Bootstrap();
        bootstrap
                // 2.1. 绑定线程组
                .group(workGroup)
                // 2.2. 指定NIO模式, 因为是Client端, 所以要绑定NioSocketChannel
                .channel(NioSocketChannel.class)
                // 2.3. 进行初始化ChannelInitializer , 用于构建双向链表 "pipeline" 添加业务handler处理
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        // 2.3.1. 测试特殊字符方式解决TCP拆包/粘包问题, 以"$_"结尾作为分割符(只要收到$_,就认为到这里就是一个数据包), 最大帧长1024bit
                        ByteBuf delimiterBuf = Unpooled.copiedBuffer("$_".getBytes());
                        ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, delimiterBuf));

                        // 2.3.2. Netty String解码器
                        ch.pipeline().addLast(new StringDecoder());

                        // 2.3.2. 配置自定义具体业务接收和处理的方法
                        ch.pipeline().addLast(new Pkg2NettyClientHandler());
                    }
                });

        // 3. 使用辅助配置工具类绑定要监听的端口, 并启动服务 => 同步阻塞
        ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 8765).syncUninterruptibly();

        // 4. 打破连接的同步阻塞, 则发送一条数据到服务器端 => 循环发送100次带特殊字符的消息
        for (int i = 0; i < 100; i++) {
            channelFuture.channel().writeAndFlush(Unpooled.wrappedBuffer(("消息" + i + "$_").getBytes()));
        }

        // 5. 阻塞的方式等待通道关闭 => 如果两边都不关闭, 则客户端和服务端的通道都会一直开着
        channelFuture.channel().closeFuture().sync();

        // 6. 释放资源
        workGroup.shutdownGracefully();
    }
}
```

###### 4、ClientHandler

```java
/**
 * 测试Netty: 客户端业务处理器, 测试特殊字符方式解决TCP拆包/粘包问题
 */
public class Pkg2NettyClientHandler extends ChannelInboundHandlerAdapter {

    /**
     * 通道激活方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty client channel active... ");
    }

    /**
     * 通道关闭方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty client channel inactive... ");
    }

    /**
     * 读写数据核心方法 => 用于处理服务端回写的数据
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 固定模式的 try .. finally
        // 在try代码片段处理逻辑, finally进行释放缓存资源, 也就是 Object msg (buffer)
        try {
            // 1. 使用了Netty String解码器后, 可以直接转换成String类型
            String response = (String) msg;

            // 5. 构造响应体给客户端 => 测试与客户端的交互
            System.err.println("Netty client: " + response);
        } finally {
            // 6. 释放资源
            ReferenceCountUtil.release(msg);
        }
    }

    /**
     * 读写数据完毕方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        System.err.println("Netty server channel read complete... ");
    }

    /**
     * 捕获异常方法
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        // 客户端读写异常, 则关闭连接
        ctx.close();
    }
}
```

##### 3）Netty 报文帧解决方案 | LengthFieldBasedFrameDecoder

###### 1、NettyServer

```java
public class Server {
    
	private static void start(int port) throws InterruptedException {
		EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup);
            b.channel(NioServerSocketChannel.class);
            b.childHandler(new ChannelInitializer() {
				@Override
				protected void initChannel(Channel ch) throws Exception {
					 ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(1024,0,2,0,2));
					 ch.pipeline().addLast(new DefaultEventExecutorGroup(16), new IProtocalHandler());
					 ch.pipeline().addLast(new StringEncoder(CharsetUtil.UTF_8));
				}            	
            });
            
            ChannelFuture f = b.bind(port).sync();
            f.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
	}
	
    public static void main( String[] args ) throws InterruptedException{
    	 start(8084);
    }
}
```

###### 2、NettyServerHandler

```java
public class IProtocalHandler extends ChannelInboundHandlerAdapter {
	@Override
	public void channelRead(ChannelHandlerContext ctx, final Object msg) throws Exception {
		int sleep = 500 * new Random().nextInt(5);
		System.out.println("sleep:" + sleep);
		Thread.sleep(sleep);
		
		final ByteBuf buf = (ByteBuf) msg;
		String outputStr = buf.toString(CharsetUtil.UTF_8);
		System.out.println(outputStr);
		
		ctx.channel().writeAndFlush(outputStr+"\n");
	}
}
```









