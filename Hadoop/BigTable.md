# 1. BigTable

## 1.1. 介绍

* BigTable和数据库类似
* BigTable不支持完整的关系数据模型，它提供了一个简单的数据模型，利用这个模型，客户可动态控制数据的分布和格式
* 数据的下标是行和列的名字,名字可以是任意的字符串

## 1.2. 数据模型

**BigTable** 是一个稀疏、分布式、持久化存储的多维度排序Map。

```hadoop
(row:string, column:string,time:int64)->string
```

例子：存储海量的网页及相关信息

使用**URL**作为行关键字，网页的某些属性作为列名，内容放在**contents**列中，网页的时间截作为标识（获取不同时间下的网页）。

* **行**：可以是任意字符串（64KB），BigTable通过行关键字的字典顺序组织数据，表中每**行**都可动态分区，每个分区叫做一个 **"Tablet"** ,**Tablet** 是数据分布和负载均衡调整的最小单位

* **列族**：列关键字组成的**集合**叫做 **"列族"**，列族是访问控制的基本单位。列族不能太多，列可以无限，eg. anchor:cnnsi.com、anchor:my.look.ca，anchor为列族

* **时间戳**：在Bigtable中,表的每一个数据项都可以包含同一份数据的不同版本，不同版本的数据通过时间戳来索引，Bigtable时间戳的类型是**64位整型**，精确到毫秒

## 1.3. API

```c++
// 打开Table
Table *T = OpenOrDie(“/bigtable/web/webtable”);
// 添加新的anchor,删除旧的anchor
RowMutation r1(T, “com.cnn.www”);
r1.Set(“anchor:www.c-span.org”, “CNN”);
r1.Delete(“anchor:www.abc.com”);
Operation op;
Apply(&op, &r1);
```

## 1.4. BigTable构件

* **GFS**：Google的分布式文件系统存储日志和数据文件

* **SSTable**：BigTable内部存储数据的文件格式，SSTable是一个**持久化的**、**排序的**、**不可更改**的**Map结构**。从内部看,SSTable是一系列的数据块(通常每个块的大小是**64KB**,这个大小是可以配置的)。SSTable使用块索引(通常存储在**SSTable的最后**)来定位数据块

* **Chubby**：一个高可用的、序列化的**分布式锁服务**组件，任务：确保在任何给定的时间内最多只有一个活动的Master副本、存储BigTable数据的自引导指令的位置、存储BigTable数据的自引导指令的位置、存储BigTable数据的自引导指令的位置、存储BigTable数据的自引导指令的位置


## 1.5. 介绍

BigTable三个主要的组件：

1. 链接到客户程序中的库
2. 一个Master服务器和多个Tablet服务器
3. 针对系统工作负载的变化情况，BigTable可以动态的向集群中添加（或删除）Tablet服务器

Master服务器主要负责工作：

1. 为Tablet服务器分配Tablets
2. 检测新加入的或者失效的Tablet服务、对Tablet服务器进行负载均衡
3. 垃圾收集、处理相关的修改操作

### 1.5.1. Tablet的位置

三层的、类似B+树的结构存储Tablet的**位置信息**

1. 第一层是一个存储在**Chubby**中的文件，包含了Root Tablet的位置信息
2. 第二层Root Tablet，它包含了一个特殊的**METADATA**表记录了所有的Tablet的位置信息，且Root Tablet不会被分割，保证存储结构不超三层，Root Tablet实际上是**METADATA**表的第一个Tablet
3. 第三层**METADATA**，Tablet的位置信息存放在行关键字下，而这个行关键字是由Tablet所在的表的标识符和Tablet的最后一行编码而成的。

若**METADATA**的每一行大约1KB的内存数据，在一个容量限制为128MB的METADATA Tablet中，可标识2^34个Tablet地址
(Root Tablet: 2^17， 一个metadata tablet 128mb = 2^17)

### 1.5.2. Tablet分配

* 在任何一个时刻,一个Tablet只能分配给一个Tablet服务器
* BigTable使用Chubby跟踪记录Tablet服务器的状态，当一个Tablet服务器启动时,它在Chubby的一个指定目录下建立一个有唯一性名字的文件,并且获取该文件的独占锁
* Master服务器负责检查一个Tablet服务器是否已经不再为它的Tablet提供服务了,并且要尽快重新分配它加载的Tablet

当集群管理系统启动了一个Master服务器：
1. Master服务器从Chubby获取一个唯一的Master锁, 用来阻止创建其它的Master服务器实例
2. Master服务器扫描Chubby的服务器文件锁存储目录, 获取当前正在运行的服务器列表
3. Master服务器和所有的正在运行的Tablet表服务器通信,获取每个Tablet服务器上Tablet的分配信息
4. Master服务器扫描METADATA表获取所有的Tablet的集合。在扫描的过程中,当Master服务器发现了一个还没有分配的Tablet,Master服务器就将这个Tablet加入未分配的Tablet集合等待合适的时机分配

当METAFATA表的Tablet还未被分配，先分配。

现有Tablet的集合只有在以下事件发生时才会改变：
* 建立了一个新表或删除一个旧表
* 两个Tablet被合并了、或者一个Tablet被分割成两个小的Tablet

### 1.5.3. Tablet服务
涉及部分：
1. GFS: 分布式文件存储系统，存储持久化状态信息（tablet log。SSTable Files）
2. memtable: 在内存中的缓存，用于存储最近提交的信息
3. SSTable：文件，存储较早提交的信息

* 当对Tablet服务器进行写操作时,Tablet服务器首先要检查这个操作格式是否正确、操作发起者是否有执行这个操作的权限
* 当对Tablet服务器进行读操作时,Tablet服务器会作类似的完整性和权限检查

### 1.5.4. Compactions
**Minor Compaction：**

随着写操作的执行,**memtable**的大小不断增加。当memtable的尺寸到达一个门限值的时候,这个memtable就会**被冻结**,然后创建一个新的memtable;被冻结住memtable会被转换成**SSTable**,然后写入GFS

该操作的目的：
1. 减少Tablet服务器使用的内存（**shrink**）
2. 减少必须从提交日志里读取的数据量

**Major Compaction：**

每一次Minor Compaction后都会创建一个新的**SSTable**文件，可能会造成有过多的**SSTable**, 通过定期合并SSTable文件，可限制文件的数量。

## 1.6. 优化

* 局部性群组
客户程序可以将**多个列族**组合成一个局部性群族。对Tablet中的每个**局部性群组都会生成一个单独的SSTable**，将通常不会一起访问的列族分割成不同的局部性群组可以提高读取操作的效率。

* 压缩
客户程序可以控制一个局部性群组的SSTable是否需要压缩;如果需要压缩,那么以什么格式来压缩。每个SSTable的块(块的大小由局部性群组的优化参数指定)都使用用户指定的压缩格式来压缩。虽然分块压缩浪费了少量空间( alex 注:相比于对整个 SSTable 进行压缩,分块压缩压缩率较低)。

很多客户程序使用了“两遍”的、可定制的压缩方式：
1. 第一遍采用Bentley and McIlroy’s方式,这种方式在一个很大的扫描窗口里对常见的长字符串进行压缩
2. 第二遍是采用快速压缩算法,即在一个16KB的小扫描窗口中寻找重复数据

* 通过缓存提高读操作的性能
为了提高读操作的性能,Tablet服务器使用二级缓存的策略，**扫描缓存**是第一级缓存,主要缓存Tablet服务器通过SSTable接口获取的Key-Value对;**Block缓存**是二级缓存,缓存的是从GFS读取的SSTable的Block

* Bloom 过滤器
我们可以使用Bloom过滤器查询一个SSTable是否包含了特定行和列的数据。对于某些特定应用程序,我们只付出了少量的、用于存储Bloom过滤器的内存的代价,就换来了读操作显著减少的磁盘访问的次数

* Commit 日志的实现
设置每个Tablet服务器一个Commit日志文件,把修改操作的日志以追加方式写入同一个日志文件,因此一个实际的日志文件中混合了对多个Tablet修改的日志记录

* Tablet恢复提速
当Master服务器将一个Tablet从一个Tablet服务器移到另外一个Tablet服务器时,源Tablet服务器会对这个Tablet做一次Minor Compaction，在卸载Tablet之前,源Tablet服务器还会再做一次(通常会很快)Minor Compaction,以消除前面在一次压缩过程中又产生的未归并的录。（两次Minor Compaction）

* 利用不变性

使用Bigtable时,除了SSTable缓存之外的其它部分产生的SSTable都是不变的,例如,当从SSTable读取数据的时候,我们不必对文件系统访问操作进行同步。memtable是唯一一个能被读和写操作同时访问的可变数据结构。为了减少在读操作时的竞争,我们对内存表采用COW(Copy-on-write)机制,这样就允许读写操作并行执行。
