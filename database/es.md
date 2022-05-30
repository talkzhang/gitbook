# es如何保证一致性

# 关于es的index、type、document

* index相当于数据库的database
* type相当于数据库的table
* document相当于数据库的数据
* field相当于数据库字段

es7.0之后，将type进行了弱化，可以理解为不存在，每个index只有一个默认的_doc的type，所以可以把index看成是table，es实例看成是database

# es的垂直扩容与水平扩容

* 垂直扩容：比如说有6台服务器，每台容纳1T数据，现在要增加到8T数据，垂直扩容就是重新购置两台服务器，每台服务器容量是2T，替换到老服务器，现在6台服务器总哦容量就是8T
* 水平扩容：新购置两台服务器，每天服务器的容量是1T，直接加入到集群里去。

一般采取水平扩容方案。

# 附：es选举算法

7.x之前es采取Bully，这种算法较简单写，大概就是在所有活着的节点中，选取 ID 最大的节点作为主节点，其中每个节点都知道其他节点的ID，缺点master假死可能会导致频繁切换；脑裂：当发生网络分区故障就会发生脑裂，就会出现双主情况，那么这个时候是十分危险的因为两个新形成的集群会同时索引和修改集群的数据，脑裂可以通过最小选举master节点的这个参数去解决。

7.x之后采用类似raft算法实现，但也并不完全按它论文的思想去实现，具体的细节我现在暂时想不起来了。。。，但是我知道7.x之后的选主要比之前的更稳定。

# es主节点的作用

主节点负责创建索引、删除索引、分配分片、追踪集群中的节点状态等工作。 Elasticsearch 中的主节点的工作量相对较轻。 用户的请求可以发往任何一个节点，并由该节点负责分发请求、收集结果等操作，而并不需要经过主节点转发。

# es是什么

es可以说是一个分布式搜索引擎，天然的支持分布式，这意味着不需要其他组件也就是不需要开发者做过多的集成就可以完成一个es集群的搭建，毕竟人家内置天然支持。

基于Lucene实现，lucene是比较复杂的算法实现，es在以此为基础提供一套restful api来实现全文检索等功能。

es有分片（shard）和r副本（eplica）的概念，使得es集群环境下更加高可用，某些节点出现故障时会自动分配其他节点代替其进行工作。

# es自动生成的id如何保证id不重复的

es采用guid算法，可以保证分布式集群环境下同一时间不同的document生成的id一定不同。

# es更新操作如何做的，为什么选择那样做？

es更新分为两种，一种是全量替换，和新增一条数据的请求时一模一样，另一种是partial update，也就是局部更新，两者执行流程从大体上来讲都出多，基本上都是将源文档标记为deleted，然后创建一条新的doc，我们使用时一般使用partial update方式来更新es内的数据。

因为全量替换时，即使只想替换一个字段的值，也要把其他非要改的字段都查询出来一起请求，而partial update只需要把要改的字段信息请求过去，由es内部去查询数据并替换局部要改的字段，相比之下partial update执行更快，因为其查询和修改都是在es内部的一个shard内完成的，同时parial update也会利用version版本号进行并发控制，这些都在其shard内部实现了，而全量替换刚才也说了必须要查询所有字段的信息，相比之下增加了网络传输；另一方面，因为parial update减少了查询和修改的时间间隔，可以有效减少并发冲突的情况。

举个栗子，便于理解，比如某个index有三个字段a，b，c，在页面查询出来后用户1和用户2都会查询到这个doc，并且都想修改，用户1对a，b字段进行了修改，用户2在修改字段c，试想下，如果这个时候采用全量替换，会将a，b的原有的值又会修改到用户1未修改的值，所以说如果全量替换，在修改时需要在查询一下，然后拿到最新的数据，然后去全量替换，这就增加了网络开销，就是慢；因为慢，分成了两部操作，所以有可能发生并发冲突的情况。

# es 有哪些批量操作

* mget：查询多条数据
* bulk：同时执行多种操作，除了查询，即增删改，但是对语法要求极高，每个独立的json串必须在一行，不同操作时需要必须换行；5.0之前会将批量命令copy成一个jsonarray数组后进行路由操作，5.0之后直接对请求的批量文本解析操作，节省了大量内存；

# es为什么不能修改主分片数量？有什么解决方案？

这得从es的路由原理开始说起，首先es在索引创建时就指定了主分片数量和副本分片数量，在数据插入时，es有自己的算法来决定该数据落到哪个分片上，它是有规则的，shard = hash(routing) % number_of_primary_shards，大概就是通过一个路由的值（默认是文档id）经过hash函数得到一个hash值，然后模主分片数量，得到的这个余数就是要记录的分片的位置，这个路由值是可以在新增文档时指定的，比如可以方便按类别进行存储，不指定的话默认就是文档id，因为这种机制的存在，所以如果你修改了主分片数量就会导致数据找不到。

但是随着业务上升，数据量增加，开始设置的主分片数量不足以水平扩容了，那这时可以考虑新建一个索引将现有索引的数据全部迁移过去，然后把现有的索引删除。

# 什么是deep paging？为什么要避免deep paging？

deep paging简单说就是搜索深度过深，比如有60000条数据，有三个主节点，一个副本节点，每个节点上分布了20000条数据，如果查询按每页10条，当搜到1000页的时候，查询的其实是10000-10010区间的数据，这时它会怎么查询呢？ 大致流程如下：

客户端发起请求到某个节点，该节点此时称之为协调节点，它会从每个shard中去找10010条数据，然后得到30030条数据，然后对其进行排序后取最后10条，所以搜索过深的时候，就需要在协调节点保存大量的数据，还要进行大量的排序，排序之后，再取出对应的那一页，这个过程，非常耗费网络带宽、内存、cpu，所以deep paging的性能问题，应该尽量避免出现这种操作。

# 附：什么是mapping

（1）往es里面直接插入数据，es会自动建立索引，同时建立type以及对应的mapping  
（2）mapping中就自动定义了每个field的数据类型  
（3）不同的数据类型（比如说text和date），可能有的是exact value，有的是full text  
（4）exact value，在建立倒排索引的时候，分词的时候，是将整个值一起作为一个关键词建立到倒排索引中的；full text，会经历各种各样的处理，分词，normaliztion（时态转换，同义词转换，大小写转换），才会建立到倒排索引中  
（5）同时呢，exact value和full text类型的field就决定了，在一个搜索过来的时候，对exact value field或者是full text field进行搜索的行为也是不一样的，会跟建立倒排索引的行为保持一致；比如说exact value搜索的时候，就是直接按照整个值进行匹配，full text query string，也会进行分词和normalization再去倒排索引中去搜索  
（6）可以用es的dynamic mapping，让其自动建立mapping，包括自动设置数据类型；也可以提前手动创建index和type的mapping，自己对各个field进行设置，包括数据类型，包括索引行为，包括分词器，等等  

mapping，就是index的type的元数据，每个type都有一个自己的mapping，决定了数据类型，建立倒排索引的行为，还有进行搜索的行为

# es中filter和query的区别

filter，仅仅只是按照搜索条件过滤出需要的数据而已，不计算任何相关度分数，对相关度没有任何影响   
query，会去计算每个document相对于搜索条件的相关度，并按照相关度进行排序  

# es的写一致性原理是什么

es有一套机制，consistency的机制，就是在写入时，可以指定shard活跃的模式，总共有one（primary shard），all（all shard），quorum（default）三种策略

one：要求我们这个写操作，只要有一个primary shard是active活跃可用的，就可以执行；
all：要求我们这个写操作，必须所有的primary shard和replica shard都是活跃的，才可以执行这个写操作；
quorum：默认的值，要求所有的shard中，必须是大部分的shard都是活跃的，可用的，才可以执行这个写操作；

quorum是有计算公式的，公式是（（主分片数量+副本分片设置数量）/2）+1，这里看文档帖子什么的都没有理解，日后在看吧#TODO，总之quorum方式就是保证绝大多数shard可用时写入，同时这个策略下还有timeout设置，就是超过timeout设置的期限，还没有大多数shard处于正常状态，拒绝写入。

# es读取数据过程

1、客户端发送请求到任意一个node，成为coordinate node  
2、coordinate node对document进行路由，将请求转发到对应的node，此时会使用round-robin随机轮询算法，在primary shard以及其所有replica中随机选择一个，让读请求负载均衡  
3、接收请求的node返回document给coordinate node  
4、coordinate node返回document给客户端  
5、特殊情况：document如果还在建立索引过程中，可能只有primary shard有，任何一个replica shard都没有，此时可能会导致无法读取到document，但是document完成索引建立之后，primary shard和replica shard就都有了  

# es增删改流程

（1）客户端选择一个node发送请求过去，这个node就是coordinating node（协调节点）  
（2）coordinating node，对document进行路由，将请求转发给对应的node（有primary shard）  
（3）实际的node上的primary shard处理请求，然后将数据同步到replica node  
（4）coordinating node，如果发现primary node和所有replica node都搞定之后，就返回响应结果给客户端  

## es写入深入详解

![es写入流程原理](https://gitee.com/hongqigg/imgs-bed/raw/master/image/es%E5%86%99%E5%85%A5%E6%B5%81%E7%A8%8B.png)

1. 当es写入一个文档时，首先客户端随机访问一个节点作为协调节点，通过路由规则将该请求转发至需要执行的primary shrad，这时这条文档数据是在jvm内存中（因为es是java编写的，刚进来的文档肯定先在jvm内存中），也就是iindex buffer，同时也会写入translog到内存中，translog默认每5秒追加到磁盘一次，将数据放入index buffer和translog之后，会通知相关的副本分片也去执行该流程，执行完成之后返回客户端成功。
2. refresh：文档挡在jvm内存中是无法完成搜索的，同时也不能立刻写入磁盘，因为如果写入频繁的话，无形中增加了磁盘的io操作，会非常影响性能，为此，es通过利用系统缓冲，也就是filesystem cache用于暂存index buffer内生成的segment file，index buffer默认每隔一秒（或者index buffer内存占比超过设置上限，默认jvm堆内存10%）就会将生成的segment file refresh到filesystem cache中，此时的文档数据虽然没有到磁盘，但是已经可以被搜索到了，所以说es的搜索数据并不是实时，而是近实时的；
3. flush：数据老在os cache中放着也不行啊，老在里面放着太多内存就爆了，所以es还会有flush操作，具体是说默认每隔30分钟将os cache内的segment file写入（fsync）到磁盘，同时有一个commit point文件用来记录已经提交的segment file信息，同时将translog删除，创建一个空的translog文件，同时呢，每次发生flush动作会进行一次refresh操作，将index buffer中的segment file刷到os cache中，之后再进行flush。
4. merge segment：每秒产生一个segment文件对于es搜索来说会影响搜索效率和资源利用率，所以es后台会有一个进程专门用来对每秒产生的这些多个segment进行merge，所有的过程都不需要我们干涉，es会自动在索引和搜索的过程中完成，合并的segment可以是磁盘上已经commit过的索引，也可以在内存中还未commit的segment：合并的过程中，不会打断当前的索引和搜索功能，当然了segmentfile也是有上限的，每一个segment里面都会保存最多2^31个文档，merge完成之后es会对merge操作后的segment进行一次flush操作，更新磁盘commit point同时删除此次merge的零散的segment file文件；merge操作还有一个作用，就是在es中每个delete操作其实都是对将要删除的文档打一个.del的标签，同理update操作就是将原文档进行.del打标然后插入新文档，只有merge操作才会将这些已经打标的.del文件真正进行物理删除。
5. 如果发生宕机的话，理论上最多丢失translog内5s的数据，因为translog默认每隔5s追加写入到磁盘一次，当宕机重启后，es会对比commit point最后一次提交的记录，然后利用和translog中的记录重新生成segment file供搜索和后续刷盘流程。

综上所述，在实际运用中，es的具体配置还是要看具体使用的场景，比如你是用来存储日志的，那日志有什么特点，对数据的实时性没有那么高，那就可以考虑对在refresh时设置刷到os cache的时间久一些，同时index buffer内的内存上限也可设置大一些，这样依赖减少refresh动作，二来生成的segment文件相对会大一些，查询效率也更高，当然merge segment效率同时也会提升；另一方面，也可以设置translog的写入磁盘异步化提升效率等等。

# es查询流程

es查询过程是有多种模式的，分别是：

- query and fetch 向索引的所有分片（shard）都发出查询请求，各分片返回的时候把元素文档（document）和计算后的排名信息一起返回。这种搜索方式是最快的。因为相比下面的几种搜索方式，这种查询方法只需要去shard查询一次。但是各个shard返回的结果的数量之和可能是用户要求的size的n倍。
- query then fetch 这是es默认的搜索方式，这个和 query and fetch区别就是，第一次向索引所有分片发出请求后，各分片只返回排名信息和文档id，但不包含文档信息，然后协调节点通过排序后，通过id去获取所需的具体文档信息。
- dfs query and fetch 
- dfs query then fetch 

后两种模式就是在第一、二基础上增加了dfs流程，dfs其实就是在进行真正的查询之前， 先把各个分片的词频率和文档频率收集一下， 然后进行词搜索的时候， 各分片依据全局的词频率和文档频率进行搜索和排名，如果没有使用dfs，那么分片查询数据默认使用各分片内部的词频率和文档频率，不够准确；但是dfs肯定增加了es的操作，所以对比没有dfs操作的查询相对较慢；dfs query then fetch协调节点有可能和所有shard进行3次交互，效率是最低的。

# es和solr区别 为什么使用es

首先两者都是基于Lucene开发的搜索引擎，没什么好坏，适用于当下场景的才是最合适的，两者在搜索引擎方面都是佼佼者。

1. es天生的分布式存储引擎，solr需要结合zk完成集群搭建和管理
2. 当对已有数据进行检索时，solr检索速度要高于es；但实时建立索引，solr会产生io阻塞，效率低；es不阻塞，效率高。
3. es开箱即用，提供一套restful api来实现数据的操作，相比之下，solr学习成本更高些
4. solr支持通过html、pdf、json等多种索引方式、es只支持json格式

那么从学习成本、实时添加、查询海量数据的角度来看，es更适合我们，可以用es来做日志系统、监控系统、以及mysql表关系复杂且表数据量大的查询，可以通过flink将es作为结果集来进行存储后，对其index进行全文索引。

# 正排索引、倒排索引、数据库索引

- 正排索引：就是通过id去查找对应的数据。
- 倒排索引：就是通过某个字段去查找匹配的id，然后通过id找到对应数据，当然还会有关键词在文档中出现的位置position信息等。
- 数据库索引：以mysql来说，普通索引使用b+树，键值就是对应的主键id，通过该索引找到id之后，在通过id进行回表获取对应数据，但是数据库索引在使用like %在前面时索引会失效，同时组合索引要遵循最左前缀法则。

# es中为什么选择倒排索引而不选择B+树索引

因为es底层实现是Lucene，倒排素银是Lucene的数据结构。

## es对倒排索引做了哪些优化 也是为什么mysql有全文索引为什么还要用es的原因

es的数据结构使用的是倒排索引，当然mysq实现全文检索也是通过倒排索引实现的，但是相比mysql，es更适合进行全文检索，因为其内部Lucene对倒排索引的存储进行了很苛刻的内存优化。

倒排索引大致可以理解为是通过关键词反向查找文档，通过关键字找到对应的文件id数组，然后通过文档id数组去查找到对应的文档。

以下内容参考连接：[https://blog.csdn.net/leeta521/article/details/119376004](https://blog.csdn.net/leeta521/article/details/119376004)

### 什么是term

上面提到了关键词，关键词在es内叫term。

### 什么是posting list

就是上面提到的文档id集合，当然不是那么简单，还会有例如该关键字（term）出现的位置、次数等信息。

### term dictionary

文本建立的索引很多，搜索时想要找到某个索引不可以一个一个遍历，那样太浪费时间了，所以es对这这一堆索引进行了排序，查找时通过二分查找法，拍完序之后的索引们叫做term dictionary

### term index

ok，以上是es内倒排索引名词解释，为了提高查询速度，这个term dictionary肯定是应该放入内存的，毕竟内存操作速度远远高于磁盘，如果每次搜搜都去磁盘找倒排索引，磁盘io会增加耗时，放入内存的话有个问题需要解决，就是term dictionary文本信息肯定是海量的，如果每条都单独拉出来放入内存，那就撑爆了，所以又出来个新名词，term index，它是一个字典树结构，但是lucence对其做了优化，最终是以fst的形式放在内存中，关于fst比较复杂，个人也不是特别清楚，但大概知道通过使用fst可以大大节省内存开销，同时Lucene在磁盘上对这些索引保存是按块（block）来存储的，这样就可以通过内存公共前缀指定定位到某个block，从而找到对应的postingslist。

放张网图清晰理解存储及搜索的过程：

![](https://gitee.com/hongqigg/imgs-bed/raw/master/image/20220506172318.png)

以上是针对倒排索引的文本存储优化压缩的介绍，下面说说postings list的压缩

### postings list压缩介绍

文本索引信息存储解决了，还得想办法解决下postings list的信息，不能小看这些所谓的文档id，试想下，如果有千万甚至上亿的数据，通过性别去找的时候，倒排索引存储的id的量级可不一般了，所以对posting list也会有针对性的压缩技术。

#### Frame of Reference

Frame of Reference 简称for压缩技术，可以想象如果文档id的一堆数字，都按原数字存储的话相对来说比较消耗内存，for呢是通过一个增量编码后存储的方式来进行的，比如一个数组1，2，3，那么底层存储就是1，1，1，每个数字都是和前一位的差值，第一位的话默认前一位是0；当然这只是第一步，第二步就是posting list也是按block存储的，每个block内存储256个数字，并且每个block内的所有数字是按位来进行存储的，这样就把整型数字从8，16，32，64位变成1-64位灵活存储，更高效的利用存储空间。放一张网图来更清晰的理解压缩过程。

![](https://gitee.com/hongqigg/imgs-bed/raw/master/image/20220506172208.png)

### roaring bitmaps（针对filer cache）

以上介绍的是posting list在磁盘上的压缩存储过程，下面我们要说下内存中缓存的时候，posting list是如何存储的。

再说为什么要在内存中缓存posting list之前，应该先了解es他有个功能，叫做filter cache，它的作用就是可以把你在某个index上通过filter查询过的条件得到的这个posting list缓存到内存中，当有下次查询再过来时，如果filter的条件相同，那么则会直接使用缓存中的匹配的数据信息，这是es搜索上亿条数据快的原因，充分的利用内存，同时利用压缩技术减少内存的占用。

注意这里说的内存其实是jvm的堆内存，在这里提一下，es同时会对文档数据也会缓存，这份缓存可不在jvm内，它是在系统缓存，也就是file systemcache，所以filter结合系统缓存的运用可以使es支撑很大数据量的搜索速度很快。关于filter cache等下单独说，这里先说roaring bitmap的实现，以及使用fileter查询时面临的取交集的细节。

首先先说在内存中啊，posting list是个整型的数组，第一个压缩的思路就是用位的方式来表示，每个文档对应其中的一位，也就是我们常说的位图，bitmap，bitmap怎么存的呢，如下图：

![](https://gitee.com/hongqigg/imgs-bed/raw/master/image/20220506181803.png)

通过0和1存储，便于es这种数据库通过位运算进行and或者or的操作。

但是，位图有个很明显的缺点，不管业务中实际的元素基数有多少，它占用的内存空间都恒定不变。也就是说不适用于稀疏存储。业内对于稀疏位图也有很多成熟的压缩方案，lucene 采用的就是roaring bitmaps。

![](https://gitee.com/hongqigg/imgs-bed/raw/master/image/20220506182002.png)

将 doc id 拆成高 16 位，低 16 位。65536是16位的分界值，那么处理的时候就快要将值先%65536，得到的值就是低16位，如果做除运算就是得到高16位，同时按65536位单位进行了按block块来进行存储。

这里出来一个65536的概念，解决稀疏位图的话，可以得出bitmap每个block最大固定大小是65536/8/1000=8kb，也就是8kb是它的上限，为了防止稀疏位图，就将小于8kb的部分按原值存储，比如一个值占用2个字节，那么也就是8kb/2=4096，这个4096就是值的个数，如果小于4096就用原值存储，如果大于4096就用位图存储，ok这是这一块。

### 联合查询取交集

那么既然在内存中通过roaring bitmap进行了压缩存储，在使用的时候如何使用呢？

先讲简单的，如果查询有 filter cache，那就是直接拿 filter cache 来做计算，也就是说位图来做 AND 或者 OR 的计算。

如果查询的 filter 没有缓存，那么就用 skip list 的方式去遍历磁盘上的 postings list。

![](https://gitee.com/hongqigg/imgs-bed/raw/master/image/20220506183231.png)

以上是三个 posting list。我们现在需要把它们用 AND 的关系合并，得出 posting list 的交集。首先选择最短的 posting list，逐个在另外两个 posting list 中查找看是否存在，最后得到交集的结果。遍历的过程可以跳过一些元素，比如我们遍历到绿色的 13 的时候，就可以跳过蓝色的 3 了，因为 3 比 13 要小。

因为这个 FOR 的编码是有解压缩成本的。利用 skip list，除了跳过了遍历的成本，也跳过了解压缩这些压缩过的 block 的过程，从而节省了 cpu。


# es如何设置字段不被搜索，如何设置字段不参与分词

如果你不想该字段被搜索，可以在mapping里面设置index为false。这样es不会在该字段上建立索引，在该字段上搜索就会报错。

```bash
PUT myindex/
{
  "mappings": {
    "mytype":{
      "properties": {
      "name":{
        "type": "text",
        "index": false
      },
      "title":{
        "type": "text"
      }
    }
    }
    
  }
}
```

字符串 - text：用于全文索引，该类型的字段将通过分词器进行分词，最终用于构建索引

字符串 - keyword：不分词，只能搜索该字段的完整的值，只用于条件精准查询

```bash
PUT your_index 
{
    "settings" : {
      "number_of_shards": 1,//分片
      "number_of_replicas" : 0//副本
   },
    "mappings" : {
        
         "properties" : {
                "item_id" : {
                    "type" : "keyword",
                    "index": true
            }
         }
    }
}
```

# es缓存了解过吗 如何做到上亿数据秒级查询

es缓存分为以下几种：
1. request cache，分片级别的缓存，默认是关闭的，缓存的key是整个客户端请求，value就是搜索结果的序列化，比较适用于针对聚合类查询结果进行缓存，每次分片执行refresh操作时，缓存内容失效，用的不算特别多
2. filter cache，针对filter搜索条件的请求进行缓存，默认开启，该缓存的key是filter的条件，value是posting list，特点和query条件相比是不会进行分数排名，其它内容都一样，在后台进行merge的时候失效，充分利用好filter cache，可以大大提升检索速度。
3. Fielddata Cache 不清楚

# 生产es是怎么部署的什么配置

3个节点，每个节点32G的内存，512G磁盘，数据不是特别多，通过flink将20几张表做成几张维表之后，通过几张维表做成一张表落在es内供页面查询，这个是解决了页面查询时数据量大且链表过多查询慢的问题；还有一个场景，用于贷款申请的风控问题，比如日环比计算、同一区域地理位置单位时间内申请量是否超标等。