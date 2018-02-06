#YSCB使用方法

##安装

上官网下载**编译**好的压缩包(不推荐自己编译，容易出错)，下载地址参见 https://github.com/brianfrankcooper/YCSB/wiki，下载的压缩包有300多MB，如果不想下载，可以到实验室集群的17，18，19节点的/home/dyf目录下有现成的

##配置

这里只介绍测试Cassandra的部分，默认Cassandra集群已经搭建好了

### Cassandra配置

首先去Cassandra创建keyspace和table，运行Cassandra的bin目录下的cqlsh(这里需要有python2的运行环境支持，不过Ubuntu系统一般自带Python2.7)

```
create keyspace ycsb
    WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor': 3 }; ##副本数设置看自己的需求
    
USE ycsb;

create table usertable (
    y_id varchar primary key,
    field0 varchar,
    field1 varchar,
    field2 varchar,
    field3 varchar,
    field4 varchar,
    field5 varchar,
    field6 varchar,
    field7 varchar,
    field8 varchar,
    field9 varchar);
```

### YSCB配置

到YSCB的workloads目录下修改workload配置文件，这里我们的workload配置文件名字假设是cassandra-workload，下面说一下配置文件各个字段的含义，更详细的参考 https://github.com/brianfrankcooper/YCSB/wiki/Core-Properties

```
# 向数据库发送读写请求的IP，可以为多个，用英文的逗号分隔
# hosts=192.168.130.12,192.168.130.13,192.168.130.14
hosts=127.0.0.1

# 这个不用修改
workload=com.yahoo.ycsb.workloads.CoreWorkload

# 在开始测试性能之前，需要将多少条的记录提前装入数据库中
recordcount=1000

# 读写请求的操作总数
operationcount=1000

# true: 读取usertable中所有的字段
# false: 读取usertable中的一个字段
readallfields=true

# 读请求所占比例
readproportion=0.5

# 写请求所占比例
insertproportion=0.5

# 这个和上面创建的usertable紧密相关，上面usertable里面除了y_id，有field0到field9总共10个字段，这里就填10
fieldcount=10

# usertable里面field0到field9的类型是变长字符，这里就指明插入数据的时候字符长度
fieldlength=100

# GitHub的文档里面没有这个字段的说明含义，但是代码里面有这个字段，不清楚具体的含义，原文是: The distribution used to choose the length of a field
# 可以选择: uniform, zipfian, constant, histogram
# 默认是 constant
fieldlengthdistribution=uniform

# 请求数据的分布
# 可以选择: uniform, zipfian, latest
# 默认是 uniform
requestdistribution=zipfian

# 请求的并发线程数，这个值越大，数据库承受的压力就越大
threadcount=100

# 每秒客户端最大发起请求的数量
target=600

# 测试程序最大运行时间，单位:秒
maxexecutiontime=20
```

## 运行YSCB

为方便起见，把配置文件拷贝到bin目录下，配置文件的名字假设是cassandra-workload

> cd bin
> 
> ./ycsb load cassandra-cql -P cassandra-workload ## 这一步是先装载数据，只需要执行一次
> 
> ./ycsb run cassandra-cql -P cassandra-workload ## 这一步是开始测试读/写请求的性能，在load完之后可以执行多次


## Cassandra读写队列长度

可以使用nodetools工具查看

> cd bin
> 
> ./nodetool tpstats

可以看到

```
Pool Name                    Active   Pending      Completed   Blocked  All time blocked
MutationStage                     0         0      499571034         0                 0
ViewMutationStage                 0         0              0         0                 0
ReadStage                         0         0      146746269         0                 0
RequestResponseStage              0         0      494981137         0                 0
ReadRepairStage                   0         0       13104550         0                 0
CounterMutationStage              0         0              0         0                 0
MiscStage                         0         0              0         0                 0
CompactionExecutor                0         0         137680         0                 0
MemtableReclaimMemory             0         0            990         0                 0
PendingRangeCalculator            0         0              8         0                 0
GossipStage                       0         0         794601         0                 0
SecondaryIndexManagement          0         0              0         0                 0
HintsDispatcher                   0         0            118         0                 0
MigrationStage                    0         0             19         0                 0
MemtablePostFlush                 0         0           1061         0                 0
ValidationExecutor                0         0              0         0                 0
Sampler                           0         0              0         0                 0
MemtableFlushWriter               0         0            990         0                 0
InternalResponseStage             0         0         346310         0                 0
AntiEntropyStage                  0         0              0         0                 0
CacheCleanupExecutor              0         0              0         0                 0
Native-Transport-Requests         0         0      523437275         0            567742

Message type           Dropped
READ                         0
RANGE_SLICE                  0
_TRACE                       0
HINT                         0
MUTATION                 77391
COUNTER_MUTATION             0
BATCH_STORE                  0
BATCH_REMOVE                 0
REQUEST_RESPONSE             0
PAGED_RANGE                  0
READ_REPAIR                291

```
