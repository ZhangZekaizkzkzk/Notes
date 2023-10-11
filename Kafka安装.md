**Kafka安装**
1. 上传文件至虚拟机 /opt/software
2. 解压文件至 /opt/module
```
tar -zxvf kafka_2.12-3.3.1.tgz -C /opt/module/
```
3. 编辑配置文件 kafka_2.12-3.3.1/config/server.properties
```
#broker.id属性在kafka集群中必须要是唯一
broker.id=0
#kafka部署的机器ip和提供服务的端口号
listeners=PLAINTEXT://localhost:9092   
#kafka的消息存储文件
log.dir=/usr/local/data/kafka-logs
#kafka连接zookeeper的地址
zookeeper.connect=localhost:2181
```
4. 启动kafka(需提前启动zookeeper)
```
./kafka-server-start.sh ../config/server.properties &
```
kafka成功启动