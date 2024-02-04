# CTO问我，调研了大半年，Iceberg表究竟能不能替换Hive表-今日头条
最近一年，Iceberg可谓是出尽了风头。Iceberg社区也一直在鼓吹Iceberg表格式必将取代Hive的主宰地位。现在Iceberg提供了Spark拓展工具，这个工具可以完全不用挪动Hive原始的ORC/Parquet文件，可以直接生成Iceberg的metadata，然后就可以拿到一个Iceberg表格。原来访问Hive表的Spark、Hive、Presto作业，切换到Iceberg表上访问完全兼容，之前SQL怎么写的，现在SQL依然可以复用。

最近CTO、CIO也跟我们提，“听说你们已经调研了将近1年的Iceberg，做过不少切换的实验，你能不能跟我说说Iceberg究竟好在哪？现在Hive表已经是数据仓库通用的存储方式，我看见很多公司在做Hive表到Iceberg的升级替换，你写个文档，找个时间汇报一下，我们要评估这个迁移必要性有哪些？

如果要讨论清楚这个问题，我们需要熟悉一下Hive主要能处理哪些事情，哪些不足，Iceberg是如何解决这些问题的。

在2010年前后，Hadoop计算平台已经能满足互联网大数据的许多需求，但是对许多非专家型的大数据从业人员来说，在易用性方面仍然存在很多不足：

1.  任何想要使用数据的用户，必须弄清楚如何将他们的问题融入到MapReduce计算模型，然后用Java代码来实现。
2.  没有元数据的定义和数据集收集信息。

为了解决问题1，需要更简单、更通用的编程模型和语言--SQL，将用户编写的Hive SQL转换成MapReduce作业，来实现响应的需求。

为了解决问题2，定义了Hive表格式，并且从那时起成为了事实上的标准。

让我们再来仔细看看Hive的表格式，这是一个位于数据仓库之上的关系层，旨在大规模地向非技术专家人员普及。

在Hive表格式中，Hive被定义成为一个或多个目录，对于单分区表是一个非分区表。使用场景最多的是由许多目录组成的分区表。

在**目录级别跟踪表**的数据，并且也是在这个级别做Hive元数据存储。分区值通过目录的方式定义：

```
/dw/hive/table/partition_column=partition_value
```

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/50a56cc20f754821bb4bcab1a1016880~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=mpujy2CsOQTFMkKQ4Na94Mo4Z%2Fk%3D)

在10年的发展历程中，Hive表作为事实的表格式标准，比如有两把刷子，有以下几个优点。

1.  可以和任意的计算引擎一同使用，自大数据技术诞生以来，它都是事实唯一的表格式标准。
2.  Hive表提供了基于分区访问和分桶存储的机制，这比全表扫描的性能要高很多。
3.  与文件的存储格式无关，运行公司或者社区开发任何更合适的文件格式（例如ORC、Parquet），并且在将数据提供到Hive表（例如Avro、Text）之前不需进行转换。
4.  Hive表提供了一个**单一中心元数据**的存储方案，对于需要和Hive表交互的整个工具生态系统来说，只需要根据元数据进行读取和输出即可。
5.  Hive的元数据方案，可以保证在**分区级别实现以原子操作**的方式更改表中数据的能力，可以实现一致的数据视图。

万事万物都有它的优劣势，Hive表格式在用于更大的数据、用户和应用程序时，许多问题也变得越来越严重。

1.  由于Hive在**分区级别实现事务的原子性**，因此可以以事务的方式添加和删除分区。但是，在文件跟踪级别不能提供事务支持，因此就**不能在文件级别实现事务的增加和删除操作**。
2.  通常的解决方法是在分区级别将整个分区复制到一个新的位置，在复制分区时进行更新/删除操作，然后将元数据的分区位置更新为新位置。
3.  这种方法效率低下，尤其是在分区很大，而修改的内容很少的时候，需要完成所有分区数据的搬运，效率很低。

Hive的事务性只能保证单个分区操作的原子性，因此不能以一致的方式同时更改多个分区中的数据。即使是将文件添加到两个分区这样简单的操作，也不能以事务一致性的方式完成。因此，用户可能会查询到不一样的数据视图（脏数据），最终数据的准确性受到决策的挑战。

由于Hive自身的限制，无论是谁在写入Hive表数据时，都被严格限制在特定的标准下，遵循Hive不支持多人同时操作的问题，必须自行定义和组织协调修改顺序。多个进程同时更改数据时会导致数据丢失，因为都以最后一次写入为准，中间的操作都被最后一次操作覆盖。

因为单个分区中不能包含全部数据，因此需要做表的全量扫描才能检索到需要的数据，因此查询所有目录的数据耗时比较大。

1.  由于Hive表的统计信息都是通过异步、定期读取作业中的方式收集的，因此统计信息通常已过时。此外，收集这些统计数据需要昂贵的读取作业，需要大量地扫描和计算。
2.  由于Hive表中的统计信息通常是过时的，这会导致优化器选择了不良的执行计划，因此很多计算引擎选择了，完全忽略了Hive表中的任何统计信息。

Hive表格式的大多数问题都源于它的一个方面问题，这方面问题起初看起来相当小，但最终会产生重大后果——**表中的数据在文件夹级别进行跟踪**。

Netflix发现解决Hive表格式引起的主要问题的关键是**在文件级别跟踪表中的数据**。他们不是一个指向一个目录或一组目录的表，而是将一个表定义为一个规范的文件列表。

事实上，他们意识到文件级跟踪不仅可以解决他们使用Hive表格式遇到的问题，还可以为实现更广泛的分析目标奠定基础。因此，我们想要的表格式是这样的。

1.  Iceberg表上的读取和写入不会相互干扰。
2.  Iceberg通过Optimistic Concurrency Control提供并发写入的能力。
3.  所有写入都是原子操作。

在文件级别跟踪数据，可以更有效地对数据集进行较小的更新。

在前面章节中，我们已经学习了Iceberg的基本操作，同时知道它是一种数据湖技术的解决方案。对于一个数据湖系统，可以支持多种数据源和不同的存储格式，比如事务数据、日志、埋点数据和IOT数据等。这些数据进入流批一体化计算平台以后，需要一个结构化的方案，把数据组织放到一个存储平台上，然后供后端的数据应用进行实时或者定时的查询。

这样的数据湖生态要兼有以下特征：

1.  需要支持多种丰富的数据Schema组织。
2.  需要具有ACID保证，确保不能出现脏数据读取，同时还要满足实时性查询。
3.  对于原始数据的随时更改数据格式（增加某一列）操作，需要避免像传统的数仓一样，可能要把所有的数据重新提出来写一遍，重新注入到存储；而是需要一个轻量级的解决方案来达成需求。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/9d95369a6f214443b465afed960fc338~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=ZWoLgUjJ8aV%2B3pBA2ZKI7oj71kQ%3D)

如下图所示，最上边是命名空间，用作数据库表的隔离；中间有多个表元数据，可以提供多种数据Schema的保存；最下边的表数据用来具体存放数据，表格需要提供ACID的特性，也支持局部Schema的演进。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/8ffff137270046969780e152329e4e09~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=qz89isgOTzs6neUzWvVIhg4uBYE%3D)

记录了当前或者历史上表格Scheme的变化、分区的配置、所有快照 Manifest File 路径、以及当前快照是哪一个。同时，Iceberg 提供命名空间以及表格的抽象，做完整的数据组织管理。

存储Manifest File路径及其Partition，数据文件统计信息。链接底下多个 Manifest File，同时记录 Manifest File 对应的分区范围信息，也是为了方便后续做过滤查询；Manifest List 其实已经表示了快照的信息，它包含当下数据库表所有的数据链接，也是 Iceberg 能够支持 ACID 特性的关键保障。有了快照，读数据的时候只能读到快照所能引用到的数据，还在写的数据不会被快照引用到，也就不会读到脏数据。多个快照会共享以前的数据文件，通过共享这些 Manifest File 来共享之前的数据。

记录底下数据文件（DataFile）的路径以及每列数据的上下边界，方便过滤查询文件。

实际表内容数据，以 Parque，ORC，Avro 等文件格式保存数据。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/14ebb145adff4462b328c1863396b646~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=NG3NxpiGCeHC2UaqiBLjRh5esns%3D)

上方为Iceberg数据写入的流程图，这里以计算引擎Flink为例。

*   首先，Data Workers会从元数据上读出数据进行解析，然后把一条记录交给Iceberg存储；
*   与常见的数据库一样，Iceberg也会有预定义的分区，那些记录会写入到各个不同的分区，形成一些新的文件；

*   Flink有个CheckPoint机制，文件到达以后，Flink就会完成这一批文件的写入，然后生成这一批文件的清单，接着交给 Commit Worker；
*   Commit Worker会读出当前快照的信息，然后与这一次生成的文件列表进行合并，生成一个新的Manifest List以及后续元数据的表文件的信息，之后进行提交，成功以后就形成一个新的快照。

如下图所示，虚线框（snapshot-1）表示正在进行写操作，但是还没有发生commit操作，这时候snapshot-1是不可读的，用户只能读取已经commit之后的snapshot。同理，snapshot-2，snapshot-3表示已经可读。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/75641b05b369452e8d285dadb54d7f4a~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=80YgBM7RrijrH6Enae3Bpf%2FOIHc%3D)

可以支持并发读，例如可以同时读取S1、S2、S3的快照数据，同时，可以回溯到snapshot-2或者snapshot-3。在snapshot-4 commit完成之后，这时候snapshot-4已经变成实线，就可以读取数据了。

例如，现在current Snapshot的指针移到S3，用户对一张表的读操作，都是读current Snapshot指针所指向的 Snapshot，但不会影响前面的snapshot的读操作。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/a3e34b6f49854ce6acb9226a8563ee6e~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=ZZMiZW2QhAl5OgBe6mmBZrY3VgI%3D)

上图为Iceberg数据查询流程。

*   首先是Flink Table scan worker做一个scan，scan的时候可以像树一样，从根开始，找到当前的快照或者用户指定的一个历史快照，然后从快照中拿出当前快照的Manifest List文件，根据当时保存的一些信息，就可以过滤出满足这次查询条件的Manifest File；
*   再往下经过Manifest File里记录的信息，过滤出底下需要的Data Files。这个文件拿出来以后，再交给Recorder reader workers，它从文件中读出满足条件的Recods，然后返回给上层调用。

这里可以看到一个特点，就是在整个数据的查询过程中没有用到任何List，这是因为Iceberg完整地把它记录好了，整个文件的树形结构不需要List，都是直接单路径指向的，因此查询性能上没有耗时List操作，这点对于对象存储比较友好，因为对象存储在List上面是一个比较耗资源的操作。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/855b2df47bd44c7ba18b064584d1e5e0~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=igMA7b%2BDgT%2BN%2F0pGSTwGoV5siNA%3D)

如上图所示，Iceberg的每个snapshot都包含前一个snapshot的所有数据，每次都相当于全量读取数据，对于整个链路来说，读取数据的代价是非常高的。

如果我们只想读取当前时刻的增量数据，就可以根据Iceberg中Snapshot的回溯机制来实现，仅读取Snapshot1到Snapshot2的增量数据，也就是下图中的**紫色**数据部分。

同理，S3也可以只读取**红色**部分的增量数据，也可以读取S1-S3的增量数据。

Iceberg支持读写分离，也就是说可以支持并发读和增量读。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/50540ddafb00475ca5198a5ec2bc958f~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=HmBcsX03S9jFErdufwLwYAgI9Ig%3D)

如上图所示，Catalog 主要提供几方面的抽象。

*   它可以对Iceberg定义一系列角色文件；
*   它的File IO都是可以定制，包括读写和删除；

*   它的命名空间和表的操作 (也可称为元数据操作)，也可以定制；
*   包括表的读取 / 扫描，表的提交，都可以用 Catalog 来定制。

这样可以提供灵活的操作空间，方便对接各种底下的存储。

目前社区里面已经有的 Iceberg Catalog 实现可分为两个部分，一是数据 IO 部分，二是元数据管理部分。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/ffc2aff77f68445eb75c2988ba43870a~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=IOu1g8Hs5Eq3VkTMzObejL%2B%2FWDw%3D)

在此，我们特殊说一下HDFS的好处就是提供追加上传和原子性rename，这个优势正是Iceberg需要的。后续我们主要介绍Iceberg借助HDFS的实现。

和HIVE一样成为开放的静态数据存储标准，标准清晰，和语言无关和项目无关。

透明简单的使用，用户只需写入数据所有元数据的变更都是自动和底层序列化方式无关的，并且支持并发写。

解决存储可用性问题: 更好的schema管理方式、时间旅行、多版本回滚支持等。

每次写入都会成一个snapshot，每个snapshot包含着一系列的文件列表。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/7c51df857fe3408cbeac8d983e265301~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=dabQqef45xMyi02iXeCCsfyYBAk%3D)

基于MVCC(Multi Version Concurrency Control)的机制，默认读取文件会从最新的版本，每次写入都会产生一个新的snapshot， 读写相互不干扰。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/0d05586405ee4eac881afcd5d3c4dc6c~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=U5tYwNVdaokUPGnFOb1fc1faoQ8%3D)

基于多版本的机制可以可用轻松实现回滚和时间旅行的功能，读取或者回滚任意版本的snapshot数据。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/3c2a78f15e1d496aa1dadcce804e73a9~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=l8IvtSvTalB5eJr8c%2FBq45%2FeAmw%3D)

如上图所示，snapshot信息、manifest信息以及文件信息， 一个snapshot包含一系列的manifest List信息，每个manifest file存储了一系列的文件列表。

Iceberg写入数据文件提交snapshot的时候会依次生成manifest文件、manifestList文件以及snapshot文件，那这些元数据文件中都存放什么记录呢？

如果使用HiveCatalog创建的表，可以直接使用spark读出相关元数据记录。如果是HadoopCatalog创建的表，暂时不能直接使用spark读出相关信息，但是也可以使用文件解析器解析出相关元数据文件内容。

包含了详细的manifest列表，产生snapshot的操作，以及详细记录数、文件数、甚至任务信息，充分考虑到了数据血缘的追踪。

```
 public void run() {
        
        spark.read().format("iceberg").load("hive_iceberg.ods_iot_sensordata.snapshots").show();
    }
```

```
+--------------------+-------------------+---------+---------+--------------------+--------------------+
|        committed_at|        snapshot_id|parent_id|operation|       manifest_list|             summary|
+--------------------+-------------------+---------+---------+--------------------+--------------------+
|2021-10-09 21:58:...|8720955401832739213|     null|   append|hdfs://bigdata185...|[spark.app.id -> ...|
+--------------------+-------------------+---------+---------+--------------------+--------------------+
```

保存了每个manifest包含的分区信息。

```
 public void run() {
        
        spark.read().format("iceberg").load("hive_iceberg.ods_iot_sensordata.manifests").show();
    }
```

```
+--------------------+------+-----------------+-------------------+----------------------+-------------------------+------------------------+-------------------+
|                path|length|partition_spec_id|  added_snapshot_id|added_data_files_count|existing_data_files_count|deleted_data_files_count|partition_summaries|
+--------------------+------+-----------------+-------------------+----------------------+-------------------------+------------------------+-------------------+
|hdfs://bigdata185...|  5899|                0|8720955401832739213|                     1|                        0|                       0|                 []|
+--------------------+------+-----------------+-------------------+----------------------+-------------------------+------------------------+-------------------+ 
```

保存了每个文件字段级别的统计信息，以及分区信息。

```
 public void run() {
        
        spark.read().format("iceberg").load("hive_iceberg.ods_iot_sensordata.files").show();
    }
```

```
+-------+--------------------+-----------+------------+------------------+--------------------+--------------------+--------------------+----------------+--------------------+--------------------+------------+-------------+------------+-------------+
|content|           file_path|file_format|record_count|file_size_in_bytes|        column_sizes|        value_counts|   null_value_counts|nan_value_counts|        lower_bounds|        upper_bounds|key_metadata|split_offsets|equality_ids|sort_order_id|
+-------+--------------------+-----------+------------+------------------+--------------------+--------------------+--------------------+----------------+--------------------+--------------------+------------+-------------+------------+-------------+
|      0|hdfs://bigdata185...|    PARQUET|           1|               991|[1 -> 56, 2 -> 49...|[1 -> 1, 2 -> 1, ...|[1 -> 0, 2 -> 0, ...|        [3 -> 0]|[1 -> sensor_01, ...|[1 -> sensor_01, ...|        null|          [4]|        null|            0|
+-------+--------------------+-----------+------------+------------------+--------------------+--------------------+--------------------+----------------+--------------------+--------------------+------------+-------------+------------+-------------+ 
```

```
 public void run() {
        spark.read().format("iceberg").load("hive_iceberg.ods_iot_sensordata.history").show();
    }
```

```
+--------------------+-------------------+---------+-------------------+
|     made_current_at|        snapshot_id|parent_id|is_current_ancestor|
+--------------------+-------------------+---------+-------------------+
|2021-10-09 21:58:...|8720955401832739213|     null|               true|
+--------------------+-------------------+---------+-------------------+
```

如此完善的统计信息，利用查询引擎层的条件下推，可以快速的过滤掉不必要文件，提高查询效率，熟悉了Iceberg的机制，在写入Iceberg的表时按照需求以及字段的分布，合理的写入有序的数据，能够达到非常好的过滤效果。

在Iceberg的实际的存储文件中，schema的内容都是id，读取时和上图的元数据经过整合生成用户想要的schema，利用这种方式Iceberg可以轻松的做的column rename，数据文件不需要修改的目录，且历史文件也能够完美的兼容的新的schema。

```
{
  "format-version" : 1,
  "table-uuid" : "ded8c13f-e770-4b9f-8703-264252cb5ca6",
  "location" : "hdfs://bigdata185:8020/spark/iceberg/warehouse/hadoop_iceberg/ods_iot_sensordata",
  "last-updated-ms" : 1633789629398,
  "last-column-id" : 3,
  "schema" : {
    "type" : "struct",
    "schema-id" : 0,
    "fields" : [ {
      "id" : 1,
      "name" : "sensorid",
      "required" : true,
      "type" : "string"
    }, {
      "id" : 2,
      "name" : "ts",
      "required" : true,
      "type" : "long"
    }, {
      "id" : 3,
      "name" : "temperature",
      "required" : true,
      "type" : "double"
    } ]
  },
  "current-schema-id" : 0,
  "schemas" : [ {
    "type" : "struct",
    "schema-id" : 0,
    "fields" : [ {
      "id" : 1,
      "name" : "sensorid",
      "required" : true,
      "type" : "string"
    }, {
      "id" : 2,
      "name" : "ts",
      "required" : true,
      "type" : "long"
    }, {
      "id" : 3,
      "name" : "temperature",
      "required" : true,
      "type" : "double"
    } ]
  } ],
  "partition-spec" : [ ],
  "default-spec-id" : 0,
  "partition-specs" : [ {
    "spec-id" : 0,
    "fields" : [ ]
  } ],
  "last-partition-id" : 999,
  "default-sort-order-id" : 0,
  "sort-orders" : [ {
    "order-id" : 0,
    "fields" : [ ]
  } ],
  "properties" : {
    "write.format.default" : "PARQUET"
  },
  "current-snapshot-id" : 6590880101585303400,
  "snapshots" : [ {
    "snapshot-id" : 6590880101585303400,
    "timestamp-ms" : 1633789629398,
    "summary" : {
      "operation" : "append",
      "spark.app.id" : "local-1633789619971",
      "added-data-files" : "1",
      "added-records" : "1",
      "added-files-size" : "991",
      "changed-partition-count" : "1",
      "total-records" : "1",
      "total-files-size" : "991",
      "total-data-files" : "1",
      "total-delete-files" : "0",
      "total-position-deletes" : "0",
      "total-equality-deletes" : "0"
    },
    "manifest-list" : "hdfs://bigdata185:8020/spark/iceberg/warehouse/hadoop_iceberg/ods_iot_sensordata/metadata/snap-6590880101585303400-1-61f0cfb7-7d03-4430-bd4c-ac712b949ddb.avro",
    "schema-id" : 0
  } ],
  "snapshot-log" : [ {
    "timestamp-ms" : 1633789629398,
    "snapshot-id" : 6590880101585303400
  } ],
  "metadata-log" : [ {
    "timestamp-ms" : 1633789621668,
    "metadata-file" : "hdfs://bigdata185:8020/spark/iceberg/warehouse/hadoop_iceberg/ods_iot_sensordata/metadata/v1.metadata.json"
  } ]
}
```

Iceberg的设计本身不受底层文件格式限制，目前支持avro、orc、parquet等文件格式， 本身parquet的元数据也包含了很多和Iceberg类似的精准的统计元信息，在数据量较小时，Iceberg提升不会特别明显,甚至没有提升，Iceberg比较适合超大数据量的表。

在数据仓库和数据湖建设过程中，表通常有三种操作模式：

1.  **入湖/仓**：通过Spark、Flink把Kafka中的数据导入到湖/仓。
2.  **批作业分析湖/仓的数据**：批量跑Spark/Hive/Presto/Flink作业，做聚合或者统计，呈现报表。

3.  **出湖/仓**：一般是把湖/仓中的数据，通过批作业做一些过滤、清洗、轻度聚合，导出到下游表。

![](https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/b20e5835e10b4552919e06f10719c396~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1707624491&x-signature=D5s99WOPvCTsELQCBiS1%2FOPyArM%3D)

相比Hive数仓，Iceberg数据湖的近实时优势主要体现在以下几个方面：

在使用Iceberg作为数据湖的解决方案时，我们可以保证数据在10分钟以内写入一次，相比Hive数仓的30分钟最小周期提升了一大块。本质在于Iceberg的元数据并不保存在Hive-MetaStore中，而是直接托管在HDFS上，不存在大表扩展的问题。

数据以10分钟的频率入湖，必然会产生很多小文件，尤其是会产生大量的Manifest File，这会严重影响查询效率和性能。

但是，我们可以通过后台的程序合并Iceberg的data小文件和元数据小文件。在这种机制下，10分钟频率的入湖并不会比1小时一次的频率入湖多出太多的Metadata文件。

在Hive数仓中，写操作通常要先写到临时文件，最后再一次性Move多个临时文件到Hive目录中，而同时Move文件并不保证数据的原子性操作。因此，Hive的常规做法是每次写入都写入一个新的Partition，然后查询的时候指定Partition范围只查询旧的分区。这套体制，在T+1的离线数据分析中没有太大问题，一旦到了10分钟级别的数据实效性，数据分析师的心智负担大大增加（会不会有脏数据）。

正是有了Iceberg表的ACID机制，就不存在Hive数据更新的问题了，只要数据经过Snapshot commit之后，数据才可见，否则均不可见。**彻底解决了脏数据读取的问题**。

由于Iceberg提高了数据如何的时效性，数据入湖早，看到的数据就越及时；并且数据查询的性能也提高了优化效率。

在Hive中，每次查询都需要把Hive-Metastore里面的Partition列表拿出来，然后依次遍历每个Partition的ORC/Parquet文件，再依次把每个ORC/Parquet文件的统计信息跟查询条件对比，看看命中哪些文件，再去做数据扫描。

对于Iceberg表，每个数据文件的所有列的MAX、MIN、COUNT等统计信息都记录在Manifest file中。一个批量统计分析查询，只需要扫描Manifest File中文件，即可找到候选数据文件。

也就是说，Iceberg在统计分析这一块，比Hive做的要好很多，即使是扫描大体量的数据也能得到比较好的性能。

我们通常需要对数据仓库的数据进行分层：ODS->DWD->DWS，一份离线的T+1任务，难免会出现数据延迟。

Iceberg能做到10分钟级别的入湖操作，使得下游在等待10分钟以后，就可以跑下游数据，因此数据的时效性远远优于Hive数仓。

Hive数仓是为分析静态数据而设计的，而实际上在仓/湖上面做数据变更是业务非常常见的一个需求。而Iceberg采用的是“读取型Schema”，背后潜在的逻辑是，认为业务的不确定性是常态：既然我们无法预测业务的发展变化，那么我们就保持一定的灵活性。将结构化设计延后，让整个基础设施具备使数据“按需”贴合业务的能力。因此，数据湖更适合发展、创新型企业。

Iceberg相比Hive有很大技术层面的提升，但是我的建议仍然是“**学而不用**”，等互联网大厂把坑填完了，我们再直接短平快的上马数据湖，但是我们一定要学习。