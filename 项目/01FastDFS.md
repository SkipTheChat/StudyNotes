# 1 为什么使用fastDFS

## 1.1 传统方案的劣势

> 就是将文件传入项目中:webapp/upload

**存在问题**
1.如果图片过多、过大，就会占用项目本身的资源
2.tomcat自身对大图片的处理不占优势
3.如果使用了集群，那么图片同步将非常麻烦



## 1.2 为什么要选择fastdfs来作为中间件

### 1.2.1 FastDFS优点

**1.主备Tracker服务，增强系统的可用性**

**2.FastDFS是分布式文件系统，所以满足海量存储和高可用两个特点**

**3.支持在线扩容机制，增强了系统的可扩展性**

>FastDFS采用了分组存储方式。**集群由一个或多个组构成**，集群存储总容量为集群中所有组的存储容量之和。**一个组由一台或多台存储服务器组成，同组内的多台Storage server之间是互备关系**，同组存储服务器上的文件是完全一致的。文件上传、下载、删除等操作可以在组内任意一台Storage server上进行。
>
>类似木桶短板效应，**一个组的存储容量为该组内存储服务器容量最小的那个**，由此可见组内存储服务器的软硬件配置最好是一致的。
>
>采用分组存储方式的好处是灵活、可控性较强。比如上传文件时，可以由客户端直接指定上传到的组。一个分组的存储服务器访问压力较大时，可以在该组增加存储服务器来扩充服务能力（纵向扩容）。
>
>当系统容量不足时，可以增加组来扩充存储容量（横向扩容）。采用这样的分组存储方式，可以使用FastDFS对文件进行管理，使用主流的Web server如Apache、nginx等进行文件下载。



**4.轻量级**

Tracker server在内存中记录分组和Storage server的状态等信息，不记录文件索引信息，占用的内存量很少。　FastDFS中的Storage server直接利用OS的文件系统存储文件。FastDFS不会对文件进行分块存储，客户端上传的文件和Storage server上的文件一一对应。

在FastDFS中，**客户端上传文件时，文件ID不是由客户端指定，而是由Storage server生成后返回给客户端的**。文件ID中包含了组名、文件相对路径和文件名。Storage server可以根据文件ID直接定位到文件。**因此FastDFS集群中根本不需要存储文件索引信息**，这是FastDFS比较轻量级的一个例证。

而其他文件系统则需要存储文件索引信息，这样的角色通常称作NameServer。其中mogileFS采用[MySQL](http://lib.csdn.net/base/mysql)[数据库](http://lib.csdn.net/base/mysql)来存储文件索引以及系统相关的信息，其局限性显而易见，MySQL将成为整个系统的瓶颈。



**5.不存在单点问题**

> 单点问题： 通常分布式系统采用主从模式，一个主机连接多个处理节点，主节点负责分发任务，而子节点负责处理业务，当主节点发生故障时，会导致整个系统发故障，我们把这种故障叫做单点故障。

　FastDFS集群中的Tracker server也可以有多台，Tracker server和Storage server均不存在单点问题。**Tracker server之间是对等**关系，**组内的Storage server之间也是对等关系**。
    传统的Master-Slave结构中的Master是单点，写操作仅针对Master。如果Master失效，需要将Slave提升为Master，实现逻辑会比较复杂。和Master-Slave结构相比，对等结构中所有结点的地位是相同的，每个结点都是Master，不存在单点问题。





### 1.2.2 FastDFS缺点

1、通过API下载，存在单点的性能瓶颈

2、不支持断点续传，对大文件将是噩梦

3、同步机制不支持文件正确性校验，降低了系统的可用性

4、不支持POSIX通用接口访问，通用性比较的低

5、对跨公网的文件同步，存在着比较大的延迟，需要应用做相应的容错策略





## 1.3 FastDFS为什么要结合Nginx

我们在使用FastDFS部署一个分布式文件系统的时候，通过FastDFS的客户端API来进行文件的上传、下载、删除等操作。同时通过FastDFS的HTTP服务器来提供HTTP服务。**但是FastDFS的HTTP服务较为简单，无法提供负载均衡等高性能的服务**，所以FastDFS的开发者——淘宝的架构师余庆同学，为我们提供了Nginx上使用的FastDFS模块（也可以叫FastDFS的Nginx模块）。其使用非常简单。
FastDFS通过Tracker服务器,将文件放在Storage服务器存储,但是同组之间的服务器需要复制文件,有延迟的问题.假设Tracker服务器将文件上传到了192.168.1.80,文件ID已经返回客户端,这时,后台会将这个文件复制到192.168.1.30,如果复制没有完成,客户端就用这个ID在192.168.1.30取文件,肯定会出现错误。这个fastdfs-nginx-module可以重定向连接到源服务器取文件,避免客户端由于复制延迟的问题,出现错误。



# 2 设计原理

[FastDFS](https://link.jianshu.com?t=https%3A%2F%2Fcode.google.com%2Fp%2Ffastdfs%2F)是一个开源的分布式文件系统，由tracker serverstorage server和client三个部分组成，主要解决了海量数据存储问题，特别适合以中小文件（**建议范围：4KB < file_size <500MB**）为载体。

### **1 Storage server**

Storage server（后简称storage）以组（group）为单位，一个group内包含多台storage机器，数据互为备份，存储空间以group内容量最小的storage为准，所以建议group内的多个storage尽量配置相同，以免造成存储空间的浪费。

以group为单位存储能方便的进行应用隔离、负载均衡、副本数定制，比如将不同应用数据存到不同的group就能隔离应用数据，同时还可根据应用的访问特性来将应用分配到不同的group来做负载均衡；缺点是group的容量受单机存储容量的限制，同时当group内有机器坏掉时，数据恢复只能依赖group内地其他机器，使得恢复时间会很长。

group内每个storage的存储依赖于本地文件系统，storage可配置多个数据存储目录，比如有10块磁盘，分别挂载在/data/disk1-/data/disk10，则可将这10个目录都配置为storage的数据存储目录。

storage接受到写文件请求时，会根据配置好的规则，选择其中一个存储目录来存储文件。为了避免单个目录下的文件数太多，在storage第一次启动时，会在每个数据存储目录里创建2级子目录，每级256个，总共65536个文件，新写的文件会以hash的方式被路由到其中某个子目录下，然后将文件数据直接作为一个本地文件存储到该目录中。

![](./assets/1.1.png)

1.通过组名tracker能够很快的定位到客户端需要访问的存储服务器组是group1，并选择合适的存储服务器提供客户端访问。
2.存储服务器根据“文件存储虚拟磁盘路径”和“数据文件两级目录”可以很快定位到文件所在目录，并根据文件名找到客户端需要访问的文件。



目前支持选择 group 的规则为:

1. Round robin,所有 group 轮询使用
2. Specified group,指定某个确定的 group
3. Load balance,剩余存储空间较多的 group 优先



### **2 Tracker server**

> **Tracker server作为中心结点，其主要作用是负载均衡和调度**

Tracker是FastDFS的协调者，负责管理所有的storage server和group，每个storage在启动后会连接Tracker，告知自己所属的group等信息，并保持周期性的心跳，tracker根据storage的心跳信息，建立group到[storage server list]的映射表。

Tracker需要管理的元信息很少，会全部存储在内存中；另外tracker上的元信息都是由storage汇报的信息生成的，本身不需要持久化任何数据，这样使得tracker非常容易扩展，直接增加tracker机器即可扩展为tracker cluster来服务，cluster里每个tracker之间是完全对等的，所有的tracker都接受stroage的心跳信息，生成元数据信息来提供读写服务。



### **3 上传原理**

在FastDFS中，**客户端上传文件时，文件ID不是由客户端指定，而是由Storage server生成后返回给客户端的**。文件ID中包含了组名、文件相对路径和文件名。
    Storage server可以根据文件ID直接定位到文件。**因此FastDFS集群中根本不需要存储文件索引信息**，这是FastDFS比较轻量级的一个例证。

![img](https://img-blog.csdn.net/20160724003110772?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



### **4 同步原理**

写文件时，客户端将文件写至group内一个storage server即认为写文件成功，storage server写完文件后，会由后台线程将文件同步至同group内其他的storage server。每个storage写文件后，同时会写一份binlog，binlog里不包含文件数据，只包含文件名等元信息，这份binlog用于后台同步，storage会记录向group内其他storage同步的进度，以便重启后能接上次的进度继续同步；进度以时间戳的方式进行记录，所以最好能保证集群内所有server的时钟保持同步。

storage的同步进度会作为元数据的一部分汇报到tracker上，tracke在选择读storage的时候会以同步进度作为参考。

比如一个group内有A、B、C三个storage server，A向C同步到进度为T1 (T1以前写的文件都已经同步到B上了），B向C同步到时间戳为T2（T2 > T1)，tracker接收到这些同步进度信息时，就会进行整理，将最小的那个做为C的同步时间戳。

 

###  **5 下载原理**

当客户端向Tracker发起下载请求时，并不会直接下载，而是先查询storage server（检测同步状态），返回storage server的ip和端口，
 然后客户端会带着文件信息（组名，路径，文件名），去访问相关的storage，然后下载文件。

 ![](https://img-blog.csdn.net/20160724004505121?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

 

 