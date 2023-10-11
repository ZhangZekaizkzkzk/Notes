**Flume安装**
1. 上传文件至虚拟机，目标路径 /opt/software
2. 进入software文件夹，解压文件至 /opt/module
``tar -zxvf apache-flume-1.10.1-bin.tar.gz -C /opt/module``
3. 进入module文件夹，重命名文件
```mv apache-flume-1.10.1-bin/ flume-1.10.1```
4. 由于Flume需要依赖JDK，提前看一下JDK的位置（假设本机已配置JAVA_HOME）
```
echo $JAVA_HOME
/opt/module/jdk1.8.0_212
```
5. 进入flume-1.10.1/conf，修改配置文件
```
mv flume-env.sh.template flume-env.sh
vim flume-env.sh 
```
第 22行修改为
```
export JAVA_HOME=/opt/module/jdk1.8.0_212
```
6.配置环境变量 
```
vim /etc/profile
```
添加以下内容
```
# FLUME_HOME
export FLUME_HOME=/opt/module/flume-1.10.1
export PATH=$PATH:$FLUME_HOME/bin
```
添加完成后激活环境变量
```
source /etc/profile
```
7. 测试
在flume的conf文件中创建flumejob_dir.conf文件，写入以下代码使flume监听/opt/module中的文件改动。
```
# Flume监听文件夹


# Name the components on this agent  agent别名设置
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source  设置数据源监听本地文件配置
a1.sources.r1.type = spooldir
# 监控的文件夹
a1.sources.r1.spoolDir = /opt/module
# 上传成功后显示后缀名
a1.sources.r1.fileSuffix = .COMPLETED
# 加绝对路径的文件名 默认为false
a1.sources.r1.fileHeader = true
# 忽略所有以.tmp结尾的文件（正在被写入）
# ^以任何开头 出现无限次 以.tmp结尾的文件
a1.sources.r1.ignorePattern = ([^ ]*\.tmp)

# Describe the sink  设置sink 下沉到hdfs
# 指定sink类型
a1.sinks.k1.type = hdfs
# 指定HDFS路径 %Y%m%d/%H%M%S 日期时间  ————修改项
a1.sinks.k1.hdfs.path = hdfs://hadoop102:9000/flume/testdir/%Y%m%d/%H-%M
# 上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = testdir-
#是否按照时间滚动文件夹
a1.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹 （默认30s）
a1.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个 Event 才 flush 到 HDFS 一次
a1.sinks.k1.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a1.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件 秒
a1.sinks.k1.hdfs.rollInterval = 600
#设置每个文件的滚动大小 字节（最好128M）
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与 Event 数量无关
a1.sinks.k1.hdfs.rollCount = 0
#最小冗余数(备份数 生成滚动功能则生效roll hadoop本身有此功能 无需配置) 1份 不冗余
a1.sinks.k1.hdfs.minBlockReplicas = 1

# Use a channel which buffers events in memory  设置channel  使用内存 总大小1000 每次传输100
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel  指定channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```
运行代码
```
./flume-ng agent --conf conf/ --name a1 --conf-file /opt/module/flume-1.10.1/conf/flumejob_dir.conf

```
flume成功启动