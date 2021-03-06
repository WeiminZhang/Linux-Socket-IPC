# Linux-Socket编程和进程、线程、进程间通信、线程间通信的学习总结
[相关视频](https://www.bilibili.com/video/av15185319) 
[相关博客](http://www.cnblogs.com/webor2006/category/514056.html) 

## TCP/IP的基础知识
1. TCP/IP的分层结构，(OSI分层(7层)和网际协议族分层(4层))
    * 传输层包括TCP, UDP, SCTP(流控制传输协议)
    * IP协议数据报头部(20个字节), TCP头部(20个字节), UDP头部(8个字节)
    * 异构系统传输数据的封装与解封
    * MAC地址是物理地址(每个设备是唯一的)，地址解析
    * 网际校验和
    * TCP的特点 
        * 基于字节流传输，(边界，黏包问题)
        * 面向连接，
        * 可靠传输，
        * 缓冲传输，
        * 全双工协议，
        * 流量控制 (TCP滑动窗口协议, 以字节的方式使用的，拥塞阻塞)
    * UDP的特点
        * 无连接
        * 不可靠
        * 一般情况下UDP更加高效
2. 三次握手，四次挥手

## Socket编程
1. 4层分层结构: 应用层、TCP、IP、物理层
2. 三次握手的状态(一个有11个状态)，两个队列(未完成连接队列， 已完成连接队列)
3. echo简单的回传模型，客户端和服务器端
4. 点对点聊天程序(REUSEADDR的使用，地址重复利用)
5. TCP是流协议---无边界，UDP是基于消息的协议，传输的是报文(数据包)
    * 流协议与黏包
        流协议不能确定对等方能接受多少消息(1, 1.5, 3都有可能)，read(读操作)一次接受的字节数是不确定的； 
    * 黏包产生的原因
        流量控制，拥塞阻塞，SO-SNDBUF(套接口堆的发送缓冲区), MSS大小的TCP分节，MTU大小的IP层分组
    * 黏包的处理方案
        定长包(readn, writen,会增加网络的负担)，报尾加\r\n(ftp), 包头加上包体长度, 更复杂的应用层协议
6. 僵尸进程和SIGCHLD信号
    通过捕获SIGCHLD信号避免僵尸进程的出现, 用waitpid轮询退出的子进程
7. TCP 11种状态, 三次握手，链接终止四次挥手, TIME_WAIT与SO_REUSEADDR, SIGPIPE
    收到RST段之后，如果再调用write就会产生SIGPIPE信号，signal(SIGPIPE,SIG_IGN);忽略就好
8. 五种I/O模型，select
    * 阻塞I/O
    * 非阻塞I/O
    * I/O复用(select, poll, epoll), 中心管理器
    * 信号驱动I/O
    * 异步I/O
9. 读、写、异常事件发生的条件
10. close和shutdown的区别
    * close关闭的是数据传送的两个方向
    * shutdown可以单方向发送数据
11. 套接字I/O超时设置的方法
12. 并发的一些初步知识，poll函数的使用(不受集合容量的限制)，一个进程能打开最大文件描述符的限制 
13. epoll使用, epoll与select、poll的区别，epoll的LT/ET模式
    * select 的限制: 一个进程能打开的最大文件描述符个数是有限的，FD_SIZE(fd_set)的限制
    * 一个进程能打开的最大文件描述符个数是有限的(ulimit -n number), 系统所有能打开的最大文件描述个数是有限的，跟内存大小有关
    * select和poll的共同点在于: 内核要遍历所有文件描述符，知道找到发生事件的文件描述符，(量大的话，效率会较低)
    * epoll与select、poll的区别:
        * 相比于select与poll，epoll最大的好处是它不会随着监听fd数目的增加而降低效率
        * 内核中的select和poll的实现是采用轮询来处理，轮询的fd越多，自然耗时越多
        * epoll的实现是基于回调的，如果fd有期望的时间发生就通过回调函数将其加入epoll就绪队列中，也就是说epoll只关心活跃的fd，与fd数目无关
        * 内核/用户空间 内存拷贝问题，如何让内核把fd消息通知给用户空间呢？在这个问题上select/poll采取了内存拷贝的方法，而epoll采用了共享内存的方式
        * epoll不仅会告诉应用程序有I/O时间到来，还会告诉应用程序相关的信息，这些信息是应用程序填充，因此根据这些信息应用程序就能直接定位到事件，而不必遍历整个fd集合。
    * LT/ET模式, ET模式关注的是空闲状态到就绪状态，LT模式是水平触发(触发次数更多, 效率可能更低)--内核维护的就绪队列; EAGAIN状态
14. UDP的特点
    * 无连接(没有维护端到端的状态)
    * 基于消息的数据传输服务(TCP是基于字节流的传输协议，TCP可能会出现黏包问题)，可以认为UDP的数据包之间是有边界的
    * 不可靠(数据包可能会重复，可能会丢失，还可能会乱码, 缺乏流量控制)
    * 一般情况下UDP更加高效
15. 使用UDP实现聊天室
16. UNIX域协议特点: UNIX域协议主要用于同一台主机的传输, UNIX域协议可以在同一台主机上各进程之间传递描述符，用路径来表示协议族的描述
    * UNIX域协议地址结构
    * UNIX字节流回射客户/服务
    * UNIX域套接字编程注意点; bind成功将会创建一个文件, 权限为0777&~umask
        * sun_path最好用一个绝对路径, UNIX域协议支持流式套接口与报式套接口
        * UNIX域流式套接字connect发现监听队列满时，会立即返回一个ECONNREFUSED, 这个和TCP不同, 如果监听队列满时, 会忽略到来的SYN, 这导致对方重传SYN
17. socketpair全双工的流管道的使用, sendmsg/recvmsg, UNIX域套接字传递描述符字

## 进程间的通信
1. 进程同步与进程互斥
    * 进程间通信目的
        * 顺序程序特征: 顺序性、封闭性、确定性、可再现性
        * 并发程序特征: 共享性、并发性、随机性
        * 进程通信的目的----数据传输，资源共享，通知事件，进程控制(SIGTRAP信号)
    * 进程间通信发展 (临界区，互斥资源-临界资源)
        * 管道(匿名管道只能用于父子进程或者亲缘关系的进程间进行通信)，有名管道可用于不相关的进程间通信
        * System V进程间通信
            * System V 消息队列
            * System V 共享内存
            * System V信号量
        * POSIX进程间通信(统一编程接口, Linux已经支持了System V的IPC接口)
            * 消息队列 --- 传递数据
            - 共享内存  --- 共享数据
            - 信号量   --- 同步访问和互斥访问
            - 互斥量   ---|
            - 条件变量 ---|---->POSIX标准才定义的
            - 读写锁   ---|线程
            - 套接字(socket)
    * 进程间通信分类
        * 文件
        * 文件锁 (读写锁): 写操作时(排它锁), 读操作时(共享锁) --- 互斥和同步用的
        * 管道(pipe) 和有名管道(FIFO) ---数据传输目的
        * 信号(signal) ---控制目的, 通知的目的
        * 消息队列 --- 传递数据
        * 共享内存  --- 共享数据
        * 信号量   --- 同步访问和互斥访问
        * 互斥量   ---|
        * 条件变量 ---|---->POSIX标准才定义的
        * 读写锁   ---|
        * 套接字(socket)
    * 进程间共享信息的三种方式
        * 共享的文件系统
        * 共享的内核信息
        * 共享内存区
    * IPC对象的持续性
        * 随进程持续  (pipe, FIFO)
        * 随内核持续  (System V 消息队列、共享内存、信号量)---进程结束后不会随之删除，需要显示删除或者重启电脑     
        * 随文件系统持续(POSIX消息队列、共享内存、信号量可以使用映射文件来实现) POSIX至少是随内核持续的
2. 进程通信二
    *死锁 : 死锁是指多个进程之间相互等待对方的资源, 而在得到对方资源之前又不释放自己的资源，这样造成一种循环等待的现象。如果所有进程都在等待一个不可能发生的事情, 则进程就死锁了
    * 死锁产生的必要条件
        * 互斥条件: 一段时间内某个资源仅被一个进程所占用
        * 请求和保护条件: 当进程因请求资源而阻塞时, 对已获得的资源保持不放
        * 不可剥夺条件: 进程已获得的资源在未使用完之前, 不能被剥夺, 只能在使用完时由自己释放
        * 环路等待条件: 各个进程组成封闭的环形链，每个进程都等待下一个进程所占用的资源
        * 银行家算法，哲学家就餐问题
    * 信号量与PV原语
        * 互斥: P、V在同一个进程中
        * 同步: P、V不同进程中
    * 信号量值含义
        * S>0, S表示可用资源的个数
        * S=0, 表示无可用资源，无等待进程
        * S<0, |S|表示等待队列中进程个数, -2表示有两个进程在等待
        * 信号量有一个int的记数值，还有一个等待队列
        * P操作是对信号量的记数值减一，如果value小于0，将进程置为等待状态，并加入等待队列
        * V操作是对信号量的记数值加一，如果value小于或等于0，就唤醒等待队列中的一个进程
3. System V消息队列
    * 消息队列从一个进程向另一个进程发送一块数据块(管道是基于字节流, 流管道，管道是按照先进先出的原则来进行接收的)
    * 消息是有类型的，消息与消息之间是有边界的(消息队列可以选择接收后进入的消息，有一个选项设置为负数)
    * 每条消息的最大长度有上限(MSGMAX), 消息总和不超过(MSGMNB), 系统上消息队列的总数不能超过(MSGMNI)
    * ipc对象数据结构，消息队列在内核中的表示，每条消息有一个头部+数据, 以链表的方式来组织消息队列的
    * msgsnd函数、msgrcv函数
    * 用消息队列实现回射客户/服务器程序
4. 共享内存
    * 共享内存在传递数据的时候是最快的, 共享内存区是一块特定的内存，不用一些系统调用来操作
    * mmap注意点
        * 映射不能改变文件的大小
        * 可用于进程间通信的有效地址空间完全受限于被映射文件的大小
        * 文件一旦被映射后, 所有对映射区域的访问实际上是对内存区域的访问。映射区域内容写回文件是，所写内容不能超过文件的大小
    * mmap函数(匿名内存区域--父子进程)，munmap函数，msync函数
5. 信号量
    * 信号量集结构
    * 信号量集函数
    * 信号量示例
    * 用信号量实现进程互斥示例
    * 用信号集解决哲学家就餐问题
6. 共享内存和信号量的综合应用示例
    * 用信号量解决生产者、消费者问题
    * 实现shmfifo(共享内存的先进先出的环形缓冲区)
7. POSIX消息队列
8. POSIX共享内存
9. 线程介绍
    * 什么是线程
        * 线程是一个进程内部的执行序列
    * 进程与线程
        * 进程是资源竞争的基本单位, 线程是程序执行的最小单位, 线程共享进程数据, 但也拥有自己的一部分数据
            * 线程ID
            * 一组寄存器
            * 栈
            * errno
            * 信号状态
            * 优先级
        * 进程独立的进程地址空间, 创建一个新线程要和其他线程共享同一进程的地址空间，但也会有自己的一些局部变量
    * 线程优缺点
        * 线程占用的资源要比进程少很多
        * 创建一个新线程代价小，切换快
        * 多线程能充分利用多处理器的可并行数量
        * 等待慢速I/O操作的同时，程序还可以执行其他的计算
        * 计算密集型应用和I/O密集应用为了提高性能，将I/O操作重叠, 线程池处理大量的I/O，大并发
        * 缺点：
            - 性能损失 -----同步和调度开销
            - 健壮性降低 ---- 共享进程地址空间，一个进程崩溃，其他的进程也可能崩溃(MMU)
            - 缺乏访问控制
            - 编程难度高
    * 线程模型
        * 线程调度竞争范围---进程竞争范围(同一线程中的线程竞争)和系统竞争范围(和系统的其他线程竞争)
        * N:1用户线程模型--------并没有提供线程的实现(一个线程相当于一个进程)
        * 1:1核心线程模型
        * N:M混合线程模型
    * 用线程实现echo客户、服务器
10. POSIX 信号量相关函数
11. POSIX 条件变量
    * 当一个线程互斥地访问某个变量时, 它可能发现在其他线程改变状态之前, 它什么也做不了
    * 一个线程访问队列时, 发现队列为空, 它只能等待，直到其他线程将一个节点添加到队列中，这种情况就需要用到条件变量。
12. 线程池
    * 用于执行大量短暂的任务
    * 当任务增加的时候能动态的增加线程池中的线程的数量直到达到一个阈值
    * 当任务执行完毕的时候，能动态的销毁线程池中的线程
    * 该线程池的实现本质上是生产者与消费者模型的应用。
    * 生产者线程想任务队列中添加任务，一旦队列有任务到来，如果有等待线程就唤醒来执行任务，如果没有等待线程并且线程数没有达到阈值，就创建新线程来执行。





