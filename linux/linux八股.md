[TOC]

# 文件操作相关函数

- **fcntl** 设置文件描述符属性，linux一切皆文件

  ```c
  int fd[2];
  pipe(fd);
  int flags = fcntl(fd[0], F_GETFL); 	// 获取文件属性
  flags |= O_NONBLOCK; // 添加非阻塞
  fcntl(fd[0], F_SETFL, flags);
  
  // 非阻塞就需要忙轮询
  ```

- **dup** 和 **dup2**

  ```c
  // 返回一个新的文件描述符指向oldfd指向的文件
  int dup(int oldfd);
  
  // 设置newfd指向oldfd指向的文件，多用于重定向
  int dup2(int oldfd, int newfd);
  // eg：dup2(3, 1);
  // 原来所有向1（标准输出）输出的内容，转而输入到3指向的文件中
  ```

- **lseek**

  [参考](https://zhuanlan.zhihu.com/p/470960184)

  ```c
  off_t lseek(int fd, off_t offset, int whence);
  //SEEK_SET    //参数offset即为新的读写位置。
  //SEEK_CUR    //以目前的读写位置往后增加offset个偏移量。
  //SEEK_END    //将读写位置指向文件尾后再增加offset个偏移量。
  
  // 获取文件长度
  lseek(fd, 0, SEEK_END);
  // 扩展文件长度
  lseek(fd, 10, SEEK_END);

- 文件属性操作函数

  ```c
  int access(const char* pathname, int mode);
  //pathname 是文件的路径名+文件名
  //mode：指定access的作用，取值如下，支持｜操作
  //F_OK 值为0，判断文件是否存在
  //X_OK 值为1，判断对文件是可执行权限
  //W_OK 值为2，判断对文件是否有写权限
  //R_OK 值为4，判断对文件是否有读权限
  ```

  



# 孤儿进程、僵尸进程、守护进程

- 孤儿进程

  > 父进程结束时，子进程尚未结束，此时子进程就会成为孤儿进程，孤儿进程会被init进程领养

- 僵尸进程

  > 子进程结束时，父进程不回收子进程使用的资源（如一直占用进程id），长时间堆积大量的僵尸进程会使得创建新的子进程时无可用的进程id而无法创建成功；
  >
  > 因此，父进程需要通过wait和waitpid在子进程结束时回收子进程资源

- 守护进程相关概念

  > ​	守护进程也称为精灵进程（daemon），是运行在后台的一种特殊进程。它独立于控制终端并且周期性的执行某种任务或等待某些发生的事件。守护进程是一种很有用的进程。Linux的大多数服务器就是用守护进程实现的。比如：ftp服务器，ssh服务器，web服务器http服务器等。
  >
  > Linux系统启动时会启动很多系统服务进程（守护进程），他们没有控制终端，不能直接和用户交互。其他进程都是在用户登录或运行程序时创建，在运行结束或用户注销时终止，但是守护进程不受用户登录和注销的影响，它们一直运行着。
  >
  > ps -axj命令查看系统中的进程。参数a表示不仅列当前用户的进程，也列出所有其他用户的进程；参数x表示不仅列出有控制终端的进程，也列出所有无控制终端的进程；参数j表示列出与作业控制相关的进程。
  >
  > 对比ps-aux
- 创建守护教程的步骤

  > 1、创建子进程，父进程退出。
  >
  > 经过这步以后，子进程就会成为孤儿进程(父进程先于子进程退出， 此时的子进程，成为孤儿进程，会被init进程收养)。使用fork()函数，如果返回值大于0，表示为父进程，exit(0)，父进程退出，子进程继续。
  >
  > 2、在子进程中创建新会话，使当前进程成为新会话组的组长。
  >
  > 使用setsid()函数，如果当前进程不是进程组的组长，则为当前进程创建一个新的会话期，使当前进程成为这个会话组的首进程，成为这个进程组的组长。
  >
  > 3、改变当前目录为根目录。
  >
  > 由于守护进程在后台运行，开始于系统开启，终止于系统关闭，所以要将其目录改为系统的根目录下。进程在执行时，其文件系统不能被卸下。
  >
  > 4、重新设置文件权限掩码。
  >
  > 进程从父进程那里继承了文件创建掩码，所以可能会修改守护进程存取权限位，所以要将文件创建掩码清除，umask(0)；
  >
  > 5、关闭文件描述符。
  >
  > 子进程从父进程那里继承了打开文件描述符。所以使用close即可关闭。



# IPC —— 是指在不同进程之间传播或交换信息

- 同一台主机内

  - ![](/Users/apple/Documents/work2022/C++ knowledge/picture/IPC.png)

  - （无名或匿名）管道，Unix最古老IPC

    > **半双工**，数据只能一个方向流动，具有固定的读端和写端；
    >
    > 只能使用在具有亲缘关系的进程之间通信（父子或兄弟）；
    >
    > 本质：**内核**内存中维护的缓冲区，没有文件实体，存储能力有限，不同OS大小不同；底层实际上是**环形队列**（对环形队列的认识？）；
    >
    > **字节流**，没有消息或消息边界的概念（那么**消息**和**消息边界**是什么概念，在什么场景下呢？）；
    >
    > 函数原型 `int pipe(int fd[2]);` ：fd[0] 对应读端，fd[1]对应写端；

  - FIFO，命名管道（有名管道）

    > 支持在无关进程之间交换数据；
    >
    > FIFO有路径名与之关联，在**文件系统**中以一种**特殊设备文件**形式存在；但是FIFO中的内容却存放在内存中；
    >
    > 函数原型 `int mkfifo(const char* pathname, mode_t mode);` 其中`mode`参数和`open`的`mode`参数一样指定文件相关属性；
    >
    > 一个只读的进程`open`管道被阻塞等待直到有一个只写的进程`open`管道；一个只写的进程`open`管道会被阻塞等待直到一个只读的进程`open`管道；
    >
    > **（死锁**？如何理解？：
    >
    > ​        如果写端不阻塞在open处，当进程运行到 `write` 时读端仍未 `open` 则写端进程会收到SIGPIPE信号异常终止。反正，如果读端不阻塞，当进程运行到 `read` 时写端仍未 `open` 则读端会关闭读文件描述符。
    >
    > **）**

  - 管道读写分析：

    > 所有指向管道写端文件描述符关闭（管道写端的**引用计数**为0，因为管道实体只会有一份），当有进程从pipe中读：如果有剩余数据，则剩余数据读取完后 `read` 返回 0；
    >
    > 管道写端引用计数不为0，`read` 在有数据可读时返回读到的字节数，无数据可读时阻塞；如果设置了非阻塞模式，会返回-1；（如何设置**非阻塞模式**）
    >
    > 管道读端引用计数为0，当有进程向管道 `write` 数据时会导致管道破裂，该进程会收到 SIGPIPE信号，通常会导致进程异常终止；
    >
    > 管道读端引用计数不为0，当管道可写时，返回写进去的字节数；管道满时，阻塞等待；

  - 消息队列

    > 存放在内核中，由消息队列标识符标识。消息队列克服**信号传递信息少**，**管道只能承载无格式字节流以及缓冲区大小受限**缺点。
    >
    > 以链表结构组织的一组数据；可以把消息看作一个记录，具有特定的格式以及优先级；
    >
    > 消息队列随**内核持续**，进程结束后，消息队列也会一直存在，需要调用接口显示删除或使用命令删除；每个消息队列在系统范围内对应唯一的键值；
    >
    > 消息队列可以实现消息的**随机查询**，消息不一定要以先进先出的次序读取,也可以按消息的类型读取。

  - **共享内存 shm** ---- **内存映射mmap** 什么关系？

    > 在说mmap之前，先介绍一下普通的读写文件的原理：进程调用**read**或是**write**后会陷入内核，因为这两个函数都是系统调用，进入系统调用后，内核开始读写文件，假设内核在读取文件，内核首先把文件读入自己的**内核空间**，读完之后进程在内核回归用户态，内核把读入内核内存的数据再copy进入进程的用户态内存空间。实际上我们同一份文件内容相当于读了两次，先读入内核空间，再从内核空间读入用户空间。
    >
    > **mmap在读写的时候，是如何和磁盘交互的？**
    >
    > - 磁盘文件加载在**同一块物理内存**中，不同进程映射之，但是各自的虚拟地址空间仍是独立的；
    > - 内存映射关闭映射的时候，数据写回磁盘；
    >
    > 共享内存是多个进程共享同一块物理内存；内存映射是将一块磁盘映射到每个进程的虚拟地址空间；当机器重启的时候，因为mmap把文件保存在磁盘上，所以不会丢失；但是shm就会丢失。
    >
    > 共享内存是最快的一种IPC，因为进程直接对内存进行读取；因此需要信号来保证同步问题。
  
  - **mmap**
  
    > 原型 `void* mmap(void *addr, size_t len, int prot, int flags, int fd, off_t offset);`
    >
    > 注意点：
    >
    > ​	**addr** ：映射的首地址，一般内核会自动调整，建议NULL
    >
    > ​	**len**： 映射的文件长度，必须大于0
    >
    > ​	**port**：权限，PROT_READ PROT_WRITE，必须小于文件open时指定的权限
    >
    > ​	**flags**：MAP_SHARED（进程间通信），MAP_PRIVATE;
    >
    > ​	**fd**：open打开的文件描述符
    >
    > ​	**offset**：偏移，从文件开始偏移多少字节开始映射，必须时页的整数倍（使用0即可）
    >
    > 关闭映射 `int munmap(void* addr, size_t len);`
  
  - **shm**
  
    > `int shmget(key_t key, size_t size, int shmflg);`
    >
    > 创建一个内存共享段（0填充），或获取一个已有的；
    >
    > 
    
  - 信号量
  
    > 信号量作为了一个计数器。信号量用于实现进程间的互斥与同步，而不是用于存储进程间通信数据。
    >
    > 若要在进程间传递数据需要结合**共享内存**




# IO多路复用

- 文件描述符：

  > - 进程有**用户空间**和**内核空间**
  > - 进程的PCB位于内核空间，PCB内部会管理当前进程管理的**文件描述符表**（是一个数组）
  > - **文件描述符表**内维护当前进程打开的文件：但是同时只能打开1024个
  >   - 这个是可以修改的
  >   - `ulimit -n`    查看进程文件描述符上限
  >   - `ulimit -SHn 2048`    修改进程文件描述符上限，只对当前shell有效（临时修改）
  >   - 永久修改：`vi /etc/security/limits.conf`
  > - **文件描述符表**前三个位置默认打开**标准输入、标准输出、标准错误输出**
  > - 进程重复open同一个文件，会返回不同的**文件描述符**

- select 工作原理

  > - 能够监听的文件描述符个数受限于FD_SETSIZE（一般为1024）；为什么？
  >   - 写死在内核中了，要是修改就是需要重新编译内核……
  > - 单纯改变进程打开的文件描述符个数并不能改变select监听的文件个数
  > - select采用轮询模型，容易降低服务器响应效率
  > - `int select(int ndfs, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct time_val);` 函数原型
  > - fd_set 采用的是位图的形式
  > - 首先，需要准备准备一份allset，用来备份发送给内核需要监听的文件描述符；
  > - 原因：select返回时，有监听事件发生的在位图上是1，无则为0；
  > - 其次，可以准备client[]数组，并维护最大的maxi，作为当有客户端数据到来时的轮询循环上限

- poll 工作原理

  > - 较select，需要监听的文件描述符集合和内核返回的文件描述符集合分开管理了；
  >
  > - `int poll (struct pollfd *fds, nfds_t nfds, int timeout);`     函数原型
  >
  >   - `fds` 是结构体数组，包含所有的用户需要内核监听的fd集合
  >
  > - ```c
  >   struct pollfd {
  >     int fd;
  >     short events; // 监听的事件
  >     short revent; // 监听事件中满足条件返回的事件
  >   };
  >                             
  >   // 提供了一些宏值
  >   /*
  >   POLLIN
  >   POLLOUT
  >   POLLERR
  >   */
  >                             
  >   // 用户准备一个pollfd结构体数组，里面存放需要监听的fd
  >   struct pollfd client[OPEN_MAX]; // OPEN_MAX 用户自己指定，因此可以突破1024上限
  >                             
  >   // 使用
  >   if (client[0].revents & POLLRDNORM) {
  >     ......;
  >     client[i].events = POLLRDNORM; // 指定监听读事件
  >   }
  >   ```

- epoll （底层是红黑树）

  - epoll 是Linux下多路复用IO接口select/poll的增强版本，能够显著提高程序在大量并发连接中只有少量活跃的情况下系统CPU的利用率；

  - 原因：复用文件描述符集合来传递结果而不用迫使用户每次等待事件之前必须重新准备要监听的fd集合；另一点就是获取事件时，它无须遍历整个被监听的fd集合

  - ```c
    // 1、创建epoll句柄
    int epoll_create(int size); // size 监听数目，内核会自动调整
    
    // 2、控制epoll监听的fd集合上事件：注册、修改、删除
    int epoll_ctl(int efd, int op, int fd, struct epoll_event *event);
    
    // op : 宏：EPOLL_CTL_ADD EPOLL_CTL_MOD EPOLL_CTL_DEL
    
    // event 告诉内核需要监听的事件
    // 宏 EPOLL_IN EPOLL_OUT EPOLL_ERR
    struct epoll_event{
      uint32_t events;
      epoll_data_t data;
    };
    typedef union epoll_data {
      void *ptr;
      int fd;
      uint32_t u32;
      uint64_t u64; 
    }epoll_data_t; // 注意是联合体
    
    // 3、等待所监听的文件描述符上有事件产生
    int epoll_wait(int efd, struct epoll_event *events, int maxevents, int timeout);
    
    
    // 使用
    struct epoll_event tep, ep[OPENMAX];
    tep.events = EPOLLIN; 
    tep.data.fd = listenfd;
    res = epoll_ctl(efd, EPOLL_CTL_ADD, listenfd, &tep);
    ```

  - epoll 的两种工作模式：

    - LT：水平触发

      > 只要监听的fd对应的读写队列中有数据就会出发epoll_wait

    - ET：边沿触发：

      > 只有有数据到达时才会触发，不管读写队列是否有数据；
      >
      > ET模式必须使用非阻塞的套接字，以避免由于一个文件句柄阻塞读操作把处理多个fd的任务饿死；
    
  - epoll的反应堆模型
  
    - [黑马中的反应堆源码](https://blog.csdn.net/daaikuaichuan/article/details/83862311)
  
    - `epoll_event` 结构体内`epoll_data_t data` 是一个联合体；而联合体内有一个`void *ptr` 因此，可以通过这个指针封装很多信息
  
      > ET模型
      >
      > 非阻塞
      >
      > void*ptr
    
  - epoll底层原理：红黑树，就绪队列，等待队列？
  
  - epoll事件驱动：**reactor**
  
  
  
  # 伙伴堆算法、slab
  
  [参考文章](https://www.cnblogs.com/linhaostudy/p/12445163.html)
  
  书籍：操作系统导论 **p128-p129**（简单的介绍）
  
  - 伙伴堆算法
  
    > 二分伙伴分配程序：
    >
    > 原理：
    >
    > 理解：
    >
    > 优点：
    >
    > 缺点：
  
  - slab ---- **厚块分配程序**
  
    > 出现的原因：
    >
    > 原理：
    >
    > 使用的场景：
    >
    > 理解：



# linux锁

[各种锁比较](https://baijiahao.baidu.com/s?id=1678252166115910894&wfr=spider&for=pc)

- 互斥锁

  > **API**
  >
  > ```c
  > pthread_mutex_t lock;
  > // 静态初始化 
  > pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
  > // 动态初始化
  > pthread_mutex_init(&lock, null);
  > int pthread_mutex_lock(pthread_mutex_t *mutex);
  > int pthread_mutex_unlock(pthread_mutex_t *mutex);
  > // 销毁
  > pthread_mutex_destory(&lock);
  > ```
  >
  > **应用场景**：
  >
  > 线程获取不到锁时，CPU让其睡眠，并进行线程切换，带来了消耗；
  >
  > 适合在临界区用时长的场景下；

- 自旋锁

  > **test-and-set**指令：测试并设置，返回旧值，设置新值（该指令具有原子性）
  >
  > ```c
  > int TestAndSet(int *old_ptr, int new) {
  > 	int old = *old_ptr;
  > 	*old_ptr = new;
  > 	return old;
  > }
  > 
  > // 原理
  > typedef struct lock_t {
  > 	int flag;
  > }lock_t;
  > void lock(lock_t *lock) {
  > while(TestAndSet(lock->flag, 1) == 1)
  > 	; //spin 自旋
  > }
  > ```
  >
  > 评价自旋锁：
  >
  > 1、能够保证正确性；
  >
  > 2、不保证公平性；可能会饿死
  >
  > 3、性能性：
  >
  > - 单CPU情况下，当一个持有锁的线程被CPU调度程序挂起时，另一个需要锁的线程会在CPU一个时间片内一直自旋，浪费CPU周期；
  >
  > - 多CPU情况下，如果线程数大致等于CPU数；性能还不错；
  >
  > 
  >
  > **compare-and_swap**指令：比较并交换；
  >
  > ```c
  > int CompareAndSwap(int *ptr, int expect, int new) {
  >   	int actual = *ptr;
  >   	if (actual == expect) {
  >     	*ptr = new;
  >   	}
  >   	return actual;
  > }
  > 
  > void lock(lock_t *lock) {
  >   	while(CompareAndSet(&lock->flag, 0, 1) == 1)
  >     	; //spin
  > }
  > ```
  >
  > 
  >
  > 基于**无等待同步**避免互斥：
  >
  > 使用CAS，实现一些不用加锁来保证同步问题：**无锁队列**
  >
  > 见**OS P286**
  >
  > 
  >
  > **fetch-and-add**指令：获取并增加
  >
  > 能够保证所有线程都能抢到锁；
  >
  > ```c
  > typedef struct lock_t{
  >   	int ticket;
  >   	int turn;
  > }lock_t;
  > 
  > void lock_init(lock_t *lock) {
  >   	lock->ticket = 0;
  >   	lock->turn = 0;
  > }
  > 
  > // 理解为每次取得自己的序号，并设置下一个线程的序号
  > int FetchAndAdd(int *ptr) {
  >   	int old = *ptr;
  >   	*ptr = old+1;
  >   	return old;
  > }
  > 
  > void lock(lock_t *lock) {
  >   	int myturn = FetchAndAdd(&lock->ticket);
  >   	while(lock->turn != myturn)
  >     	; //spin
  > }
  > 
  > void unlock(lock_t *lock) {
  >   	FetchAndAdd(&lock->turn);
  > }
  > ```
  >
  > **应用场景**：
  >
  > 适用于临界区执行用时短的场景下；

- 读写锁：

  > 读写锁实现不当可能会饿死写锁；
  >
  > 当有写者等待时，如何能够避免更多的读者进入并拥有锁？
  >
  > - **读优先**：当读线程先持有读锁下，写线程在获取写锁的时会被阻塞，并且在阻塞过程中，后续读线程仍然可以成功获取读锁；---- **写线程饥饿**
  >
  > - 写优先：当读线程先持有读锁时，写线程在获取写锁的时会被阻塞，并且在阻塞过程中，后续来的读线程获取读锁时会失败，于是后续读线程将被阻塞在获取读锁的操作；这样只要写线程之前的读锁释放，写线程就可以进行写操作；---- **写线程饥饿**
  >
  > - 读写公平：**用队列把获取锁的线程排队，不管是写线程还是读线程都按照先进先出的原则加锁即可，这样读线程仍然可以并发，也不会出现「饥饿」的现象。**
  >
  > **应用场景**：
  >
  > 读多写少
  
- 悲观锁：

  > 思想：认为多线程同时修改共享资源概率较高，因此访问共享资源前，先要上锁；
  
- 乐观锁：

  > 思想：认为多线程同时修改概率低；先修改共享资源，在验证这段时间是否发生冲突。
  >
  > 乐观锁全程并没有加锁，所以它也叫做无锁编程。
  
- RCU —— **Read-Copy Update**

  > - **read**：
  >
  >   读者不需要获得任何锁就可以访问RCU保护的临界区；
  >
  >   **读者在访问被RCU保护的共享数据期间不能被阻塞**；即，读者所在的CPU不能发生**线程的上下文切换**，CPU发生**进程上下文切换**称为经历一个**静默期**（quiesecnt state）；宽限期就是所有的CPU都经历一次静默期；
  >
  > - **copy**：
  >
  >   写者在访问临界区的时候，写者会先拷贝一个临界区副本，然后对副本进行修改；
  >
  > - **update**：
  >
  >   RCU机制在一个适当时机使用一个**回调函数**把指向原来临界区的指针重新指向新的被修改的临界区，**锁机制**中的**垃圾收集器**负责回调函数的调用。
  >
  >   时机：所有引用该临界区的CPU都退出对临界区的操作；即没有CPU再去操作这段被RCU保护的临界区后，这段临界区即可回收了，此时回调函数被调用；
  >
  >   **宽限期**：Grace period，例如线程删除一个节点，可以从链表中移除，但是不能直接销毁这个节点；必须等所有读线程完成以后，才能进行销毁操作；
  >
  > - **评价**
  >
  >   RCU 的读取性能的提升是在增加写入者负担的前提下完成的，因此在一个读者与写者共存的系统中，按照设计者的说法，如果写入者的操作比例在 **10%** 以上，那么久应该考虑其他的互斥方法，反正，使用 RCU 的话，能够获取较好的性能。



# 硬链接和软连接

> - 硬链接：
>
>   创建硬链接的命令：`ln file newfile`
>
>   观察可以得知：`newfile` 的inode号和 `file` inode号完全一致，文件大小完全一致；
>
>   他们底层对应的是同一份文件；修改可见；删除一个文件的时候会让inode的引用次数减一；如果引用次数位0，则删除inode对应的文件本身；
>
> - 软链接（符号链接 **Symbolic Link**）：
>
>   创建软链接的命令：`ln -s file newfile`
>
>   软链接的inode号和文件的inode号不一致；且文件大小不同；软链接创建的文件的大小，与其执行的文件的路径长度有关；
>
>   删除文件可能会导致软链接执行的文件为空；
>
>   优点：不占空间，仅仅是一个链接；



# linux下高并发优化

- **问题描述**

  `socket() failed (Too many open files)`

  > 程序打开的文件socket连接数量超过了系统的设定值；
  >
  > `ulimit -a`			查看每个用户最大允许打开的文件数量
  >
  > 默认都是1024：
  >
  > `vim /etc/security/limits.conf`		 文件内部修改重启可以生效

- **Linux高并发场景下time_wait过多的问题分析及解决**

  > 

- **使用合适的网络IO技术和I/O事件分派机制**

  [参考](https://cloud.tencent.com/developer/article/1684951)
  
  > **五种IO模型**
  >
  > - （同步）阻塞IO——BIO
  > - （同步）非阻塞IO（忙轮询）——NIO
  > - （同步）多路IO复用     **Reactor模式**
  > - （同步）信号驱动IO——AIO
  > - 异步IO    **Proactor模式**
  >
  > **信号驱动IO和异步IO的区别**：
  >
  > 信号驱动IO：
  >
  > 进程发起一个IO，会向内核注册回调函数——信号处理程序，然后进程返回（不阻塞），在内核数据就绪时就会发送一个信号给进程，进程便在信号处理程序调用IO读取数据：先将数据拷贝到用户区，在处理数据；
  >
  > 异步IO：**Proactor模式**
  >
  > 进程发起一个IO操作，进程返回（不阻塞）；内核把整个IO处理完成后，会通知进程结果。如果IO操作成功则进程直接获取到数据。



# linux系统调用





# 文件系统inode





# 进程和线程































