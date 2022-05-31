主要关注核心算法核心思想 模块之间的关系, 设计模式, 大致流程


1 node 启动
config配置 
config 里面有哪些呢 p2p rpc,ipc  进程通信 硬件钱包 websocket

存储实例, 共识引擎


2. trie  trie.go
mpt insert cache 管理lru




3. eth/handler.start():

```go
func (h *handler) Start(maxPeers int) {
	h.maxPeers = maxPeers

	// broadcast transactions
	h.wg.Add(1)
	h.txsCh = make(chan core.NewTxsEvent, txChanSize)
    // 监视交易池
	h.txsSub = h.txpool.SubscribeNewTxsEvent(h.txsCh)
	go h.txBroadcastLoop()


    // 挖矿区块广播
	// broadcast mined blocks
	h.wg.Add(1)
	h.minedBlockSub = h.eventMux.Subscribe(core.NewMinedBlockEvent{})
	go h.minedBroadcastLoop()

	// start sync handlers
	h.wg.Add(1)
	go h.chainSync.loop() // 两个fetcher 监听 一个是tx 一个是block
}
```

p2p 节点发现, 发现了以后建立加密信道(rlpx) 然后是fetcher 和downloaddoader
3. p2p
p2p/server.go

分为几块 discovery rlpx peer, queue fetcher downloader
- discovery:
主要就是维护这个结构:
table
// doRefresh 随机查找一个目标，以便保持buckets是满的。如果table是空的，那么种子节点会插入。 （比如最开始的启动或者是删除错误的节点之后）
tab.loop:
三个channel 事件: doRefresh、doRevalidate、copyLiveNodes

dorefresh -> lookupSelf -> lookup.run() -> findNodes()   
协议pingpong   


Ping：用于探测一个节点是否在线  
Pong：用于响应 Ping 命令  
FindNode：用于查找与 Target 节点异或距离最近的其他节点  
Neighbours：用于响应 FindNode 命令，会返回一或多个节点  

节点第一次启动的时候，节点会与硬编码在以太坊源码中的bootnode进行连接，所有的节点加入几乎都先连接了它。连接上bootnode后，获取bootnode部分的邻居节点，然后进行节点发现，获取更多的活跃的邻居节点


- RLPx:
    p2p/server listenloop:
    setupConn:SetupConn,这个函数执行握手协议，并尝试把连接创建位一个peer对象。


    doprotoHandshake 协议握手 就是p2p  
    协商完了做下配置  
    writeFrame  收发数据都通过这个函数进行  


    server对象主要完成的工作把之前介绍的所有组件组合在一起。 使用rlpx.go来处理加密链路。 使用discover来处理节点发现和查找。 使用dial来生成和连接需要连接的节点。 使用peer对象来处理每个连接。  
    // 启动一个listenLoop来监听和接收新的连接。 启动一个run的goroutine来调用dialstate生成新的dial任务并进行连接。 goroutine之间使用channel来进行通讯和配合。  



    - egressMac
- fetcher 
    入口: minedBroadcastLoop
	两边互相call到 有个无限循环不停 handleMessage
	然后双端发出消息后都在loop中监听, handler接受到消息也都是流转到loop里面

    双端通信协议eth66

    四个queue:

	fetcher 的四个queue:
	```
	announced:此阶段代表节点宣称产生了新的区块（这个新产生的区块不一定是自己产生的，也可能是同步了其它节点新产生的区块），Fetcher 对象将相关信息放到 Fetcher.announced 中，等待下载。

	fetching：此阶段代表之前「announced」的区块正在被下载。

	fetched：代表区块的 header 已下载成功，现在等待下载 body。

	completing：代表 body 已经发起了下载，正在等待 body 下载成功。

	queued:代表 body 已经下载成功。因此一个区块的数据：header 和 body 都已下载完成，此区块正在等待写入本地数据库。
	```


- downloader
 fetchbodies -> concurrentFetch  -> diliver 通知下载 -> Synchronise
-> syncWithPeer


Downloader.fetchHeaders:
fetchHeaders: 发送header请求，等待所有的返回。, 不断的重复这样的操作，直到完成所有的header请求。
为了提高并发性，同时仍然能够防止恶意节点发送错误的header，我们使用我们正在同步的“origin”peer构造一个头文件skeleton



 在下载前，eth 模块会选择一个 「best peer」 给 downloader 模块，所谓的 「best peer」 其实就是「TotalDifficulty」值最大的那个节点

skeleton queue