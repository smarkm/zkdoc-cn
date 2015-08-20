## Zookeeper入门指南
### Getting Started:Zookeeper分布式应用协调服务
- 先决条件
- 下载
- 独立运行
- ZooKeeper存储管理
- 连接到ZooKeeper
- ZooKeeper编程
- 运行副本ZooKeeper
- 其他优化
- --
### Getting Started:Zookeeper分布式应用协调服务
本文档能够让你快速了解是使用ZooKeeper。它主要针对的是开发商希望试试，并包含一个ZooKeeper服务器简单的安装说明，一些命令来验证它的运行，以及简单的编程实例。例如运行复制部署，优化事务日志。然而，商业部署的完整说明，请参阅[ZooKeeper管理员指南]().

###	先决条件
在管理指南中参见[系统要求]()。

### 下载
获取ZooKeeper，从Apache下载最近的一个[稳定](http://zookeeper.apache.org/releases.html)版本的镜像。

### 独立运行
建立在独立模式下一个ZooKeeper服务器很简单。该服务器包含在一个单一的罐子文件中，所以安装包括创建一个配置。

下载后解压到指定目录

开始前你需要一个配置文件。下面是一个示例，它在conf/ zoo.cfg中（默认在conf目录下有一个zoo_simple.cfg可以简单修改）：

		
	tickTime=2000  
	dataDir=/var/lib/zookeeper
	clientPort=2181
		
配置文件可以是任何名，这里为了便于讨论叫做conf/zoo.cfg。下面是每一个配置的说明：

**tickTime** ZooKeeper中使用毫秒为最基本单位，tickTime指心跳时间，默认最小的session超时时间为两倍的tickTime。

**dataDir** 存储在内存数据库快照中的位置，除非另有说明，将更新的事务日志记录到数据库中。

**clientPort** 监听客户端连接的端口。

现在，您创建的配置文件，你可以启动ZooKeeper：
	
	bin/zkServer.sh start
ZooKeeper使用log4j作为日志系统，更详细信息在参见编程者指南中的[Logging]()章节。你会看到日志消息到控制台（默认）和/或日志文件根据log4j配置。

这里列出在独立模式下运行ZooKeeper的步骤。没有副本，所有如果ZooKeeper失败则服务将会down掉。这对于大多数开发环境是好的，但如果允许ZooKeeper副本，请参见[Running Replicated ZooKeeper](#sc_RunningReplicatedZooKeeper)

### Zookeeper存储管理
对于长时间运行的生产系统ZooKeeper必须管理外部存储（datadir和日志）。关于维护方面的细节，详情见（[维护]()）。

### 连接到ZooKeeper
一旦ZooKeeper运行，你有几种方法可以连接到ZooKeeper：

- **Java**： 使用
	
	bin/zkCli.sh -server 127.0.0.1:2181

- **C**:只用在ZooKeeper源码下面的src/c中的mark cki_mt或者make cli_st编译出cli_mt(多线程)cli_st(单线程)，详见在SRC/C下的README。

	你可以从src/c下运行：
	
	LD_LIBRARY_PATH=. cli_mt 127.0.0.1:2181

	or

	LD_LIBRARY_PATH=. cli_st 127.0.0.1:2181

	这会给你一个简单的shell执行文件系统操作管理员。

一旦连接上你会看到如下信息：
	
	Connecting to localhost:2181
	log4j:WARN No appenders could be found for logger (org.apache.zookeeper.ZooKeeper).
	log4j:WARN Please initialize the log4j system properly.
	Welcome to ZooKeeper!
	JLine support is enabled
	[zkshell: 0]
从shell中键入*help*获取更多可执行的命令，如下：
	
	[zkshell: 0] help
	ZooKeeper host:port cmd args
        get path [watch]
        ls path [watch]
        set path data [version]
        delquota [-n|-b] path
        quit
        printwatches on|off
        createpath data acl
        stat path [watch]
        listquota path
        history
        setAcl path acl
        getAcl path
        sync path
        redo cmdno
        addauth scheme auth
        delete path [version]
        setquota -n|-b val path
		
现在你可以尝试一下简单的命令来感受下这个命令行接口，你可以先尝试一下 *ls*：
	
	
    [zkshell: 8] ls /
	[zookeeper]
下一步，创建一个新的znode运行*create / zk_test my_data*。这将创建一个新znode和分配新的字符串“my_data”。如下：
	
	[zkshell: 9] create /zk_test my_data
	Created /zk_test
发出另一个命令 *ls /*，看看这个目录是什么样子的：
	
	[zkshell: 11] ls /
	[zookeeper, zk_test]
请注意，zk_test目录已经被创建了。

下一步，运行get命令验证分配个znode的数据，如下：

	[zkshell: 12] get /zk_test
	my_data
	cZxid = 5
	ctime = Fri Jun 05 13:57:06 PDT 2009
	mZxid = 5
	mtime = Fri Jun 05 13:57:06 PDT 2009
	pZxid = 5
	cversion = 0
	dataVersion = 0
	aclVersion = 0
	ephemeralOwner = 0
	dataLength = 7
	numChildren = 0
我们可以通过set命令改变zk_test的数据，如下：
	
	[zkshell: 14] set /zk_test junk
	cZxid = 5
	ctime = Fri Jun 05 13:57:06 PDT 2009
	mZxid = 6
	mtime = Fri Jun 05 14:01:52 PDT 2009
	pZxid = 5
	cversion = 0
	dataVersion = 1
	aclVersion = 0
	ephemeralOwner = 0
	dataLength = 4
	numChildren = 0
	[zkshell: 15] get /zk_test
	junk
	cZxid = 5
	ctime = Fri Jun 05 13:57:06 PDT 2009
	mZxid = 6
	mtime = Fri Jun 05 14:01:52 PDT 2009
	pZxid = 5
	cversion = 0
	dataVersion = 1
	aclVersion = 0
	ephemeralOwner = 0
	dataLength = 4
	numChildren = 0 

最好，我们通过delete命令删除znode，如下：
	
	[zkshell: 16] delete /zk_test
	[zkshell: 17] ls /
	[zookeeper]
	[zkshell: 18]
现在就这样。要探索更多，继续与本文档的其余部分，并看[程序员指南]()。

### ZooKeeper编程
ZooKeeper有Java绑定和C绑定，它们功能是一致的。C绑定存在两种形式：单线程和多线程。不同的只是在如何的消息传递循环。更多信息详见[ZooKeeper程序员指南中的编程示例]().

### 运行Replicated ZooKeeper
在独立模式下运行管理员方便评价，开发，和测试。但在生产中，你应该运行在管理员复制模式。在同一个应用程序中的一个复制的服务器群被称为一个法定人数，并在复制的模式中，所有的服务器都具有相同的配置文件的副本。该文件是类似于一个用于独立模式，但有一些差异。这里就是一个例子：
	
	tickTime=2000
	dataDir=/var/lib/zookeeper
	clientPort=2181
	initLimit=5
	syncLimit=2
	server.1=zoo1:2888:3888
	server.2=zoo2:2888:3888
	server.3=zoo3:2888:3888
**initLimit** 是ZooKeeper使用超时限制ZooKeeper服务器在法定时间长度必须连接到一个leader。 The entry **syncLimit** limits how far out of date a server can be from a leader.

以上两个超时限制的时间单位都是tickTime，在上面的例子中initLimet是5 ticks，1 tick 为2000毫秒，也就是说initLimit为10秒。

*server.X*列表中列出的是提供ZooKeeper服务的Server，当Server启动，它知道它的服务器是通过寻找不存在的数据目录中的文件，该文件已经包含了服务器数量，用ASCII。

最后，请注意每个服务器名称后的端口号：“2888”和“3888”。Peers使用前一个端口连接其他的peers。这样的连接是必要的，以便Peers可以沟通，例如，同意更新的顺序。更特别的是，此端口用于连接leader和followers，当一个信息leader出现，follower通过此端口打开一个到leader的TCP连接。因为默认的leader选举也使用TCP，我们需要另一个端口来进行leader选举，这个端口就是第二个端口。

### 其他优化
有一组其他的参数可以大幅度提升性能：

- 要获得更新的低延迟是很重要的，有一个专门的交易日志目录。默认情况下，事务日志放在同一目录下的文件和一个数据快照。datalogdir参数表示一个不同的目录使用事务日志。
- [tbd: what is the other config param?]

[*译文说明*](intro.md)



