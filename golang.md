slice 扩容 如果新申请的容量是两倍还要大,就直接是申请的容量
原有slice 长度小于1024 就直接扩两倍
如果大于 就少一点


slice 为什么不是线程安全:


map的底层实现
hmap
hash 随机数种子 , tophash flag status buckets oldbuckets 扩容进度
extra 溢出桶

extra:
    overflow []bmap
    oldoverflow

bmap
tophash 快速定位是元素否在, 使用key的高8位作为 hash值,并且当hash小于一个制的时候自动加上一个数





buckets[57]:
 tophash: []
 key: []
 value:[]
 overflow
keyvalue 是两个list 如果用map来存会浪费空间


扩容 装载因子

rehash 需要把所有的维护map的数据结构 改了 tophash 会放到一个新的桶里
rehash 一个是slot 扩容, 原先的slot不够了, 这种就需要rehash, 一种是原来的桶装载因子太大, 知识多分配空间这样扩了新的 直接按序号需要移动就行


map 遍历无序就是因为rehash, 或者他其实点也有可能是随随机的


要考虑扩容

高8位 tophash中没有就去overflow中去找
低8位确定属于哪个桶

channel的接受是exclusive
多个select 会都收到关闭

channel 线程安全
chan:
buf  循环数组 不需要移动其他数据 永远从指针开始消费 维护指针
两下标 
size
阻塞队列  双向链表
lock 


channel 的接受是exclusive
select channel 收到关闭是所有都收

发送也可以select


不要使用共享内存通信 应该使用通信来共享内存


常用的并发模型 共享内存 和csp
一般的共享内存 就是ipc 共享内存加锁 go 是通过channel加gouroutine的方式来做

mutex:
type mutex:
state 
sama

state:
第1位表示是否能拿到锁
第2位是协程是否被唤醒
第三位表示是否处于饥饿模式
第4位表示排队数量

sema 就是个信号量 组色和唤醒通过这个来实现
如果让我自己设计的话 就是 加锁排队 
原子操作cas来获取锁, , 实际上使用atomic 通过信号量来实现线程阻塞与唤醒
如果获取不成功, 看是不是不是阻塞

等待时间过长会进入饥饿模式 饥饿模式会排在前面

go 互斥锁 正常模式和饥饿模式的区别


唤醒就是协程级别的阻塞
锁的阻塞有两种 一种是线程级别阻塞, 一种是gmp级别自旋



互斥锁


w
writesem
readsem
readercount
readerwait


读锁会对cas reader countcas cas +1 -1
写锁 先直接lock
写锁 先readercount 加很大的负数 防读锁
然后readerwait 

读锁写锁在加锁 lock 之前使用unlock会panic

可重入锁 获得协程ID 判断owner
有个计数 



计数 add store load cas swap 底层有硬件支持


goroutine 底层实现原理:


type g struct:
goid goroutine ID
sched gobuf
stack stack
gopc
startPC


p封装了一个队列
type p struct:
lock
id
status
runnext 优先调度


type  m strcut:
g0
curg *g


type schedt     全局一个链表


状态 idle
runable 等待m取出运行
running
syscall
waiting
dead
copystack

调度时机:
stw
系统调用
gc

runtime.newproc
 m没任务的时候会自己自旋 防止调度很快


软件级别的不也是硬件级别

创建销毁成本低,调度成本低, 不需要进入cpu的中断 ,  栈也是利用率高



拿全局队列需要加锁
每次偷一半的任务


goroutine  付么时候泄漏:

泄漏原因三类:
阻塞:channel 互相锁住
死循环
长时间等待






 
m是物理线程 ,p是软件级别的执行器, p作为物理处理器的调度单位 是基于物理线程实现的一个程序


stealwork 窃取
handoff 机制 就是p的分离机制
主动释放
抢占式调度 

编译期插入一个标记 , 让运行时有机会可以检查抢占调度
stw超过10ms就会设置抢占标记
但是只有函数调用可以抢占 没有调用额外的函数,还是不行

基于信号的抢占调度
m注册一个信号的sigup 的处理函数
然后会间隔性的进行监控 最长10ms  然后寻找其他的goroutine 运行
硬件级别的抢占调度
抢占放到全局队列里


查看运行时的调度信息:
go tool trace go tool debug

调度跟踪 goroutine 网络阻塞 同步阻塞 调度延迟


内存算法就是tcmalloc 每个线程会维护一个独立的内存池 优先从本线程那

go 管理内存的时候 把内存切的非常细

mspan mcaceh mcentral mheap

mspan是内存管理最小单元 go 自己封装的一个内存页 有非常多种规格的内存页
mcache 每个goroutine 绑定的p有一个mcache 缓存在本p的数据
mcache 管理多个 mspan
mcentral 管理全局的的mspan, 统一管理程序想操作系统申请的内存 共所有线程使用,全局mheap 包含mcentral字段,  每种central管理一种规格的mspan 将空闲的和没有空闲的分开



mcentral:
spanclass mcentral 这种规格的大小
partial  空闲的
full 非空闲的


mheap 管理所有动态的内存分配
所有中心缓存存放到mheap里面
是按照页来管理

各种缓存 中心缓存 堆 分配内存
大对象直接分配堆内存


内存逃逸: 影响gc
一旦逃逸到堆上 就会给gc带来一定的压力
编译期会根据变量是否被外部饮用来决定内存逃逸
函数外部没有引用优先放到栈上
外部有引用 就优先放到堆上
如果栈放不下  就放到堆上


变量大小不确定的时候
编译期不知道大小的类型 



内存对齐机制
实际占用大小必须是整数倍
和操作系统位数有关系
偏移变量必须是成员大小的整数倍
整个结构体的地址必须是最大字节的整数倍
结构对齐是从前向后积累 只能变大不能变小



主流gc算法
引用计数 python php swift 对象回首比较快 不会出现达到某个阈值才回收  不能很好的处理循环饮用 实时维护性能损耗

分代收集 java 根据生命周期长短划分空间 不同代有不同的算法和回收频率 算法复杂
标记清楚 需要stw

深度优先遍历标记


