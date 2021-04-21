# spring

## zookeeper

### 1.1.1 命令

>* 启动 ：bin目录下执行`sh bin/zkServer.sh start`
>* 查看状态： bin目录下执行 `sh bin/zkServer.sh status`
>* 









## kafka

### 1.1.1 命令

>* 启动命: `bin/kafka-server-start.sh -daemon config/server.properties`





## shell

### 1.1.1 命令

>* 添加权限： `chmod u+x xxx.sh`a
>* 启动sh脚本： `sh xxx.sh`

### 1.1.2 上传kafka值

```shell
#!/bin/sh
#kafka所在目录
kafkaPath=/export/servers/kafka_2.11-2.3.0
#broker
brokerlist=172.16.85.110:9092,172.16.85.111:9092,172.16.85.112:9092
#kafka的topic
topic=dingTest001
jsonObjects=('{"1xx":"1","1xxx":"11"}' '{"2xx":"2","2xxx":"22"}' '{"3xx":"3","3xxx":"33"}' '{"4xx":"4","4xxx":"44"}')
for jsonDetail in ${jsonObjects[@]}
do
 echo ${jsonDetail} | ${kafkaPath}/bin/kafka-console-producer.sh --broker-list ${brokerlist} --topic ${topic}
done
```

```shell
#!/bin/bash
kafkaPath=/opt/kafka_2.13-2.7.0
brokerlist=121.196.25.62:9092,121.196.25.62:9093,121.196.25.62:9094
topic=topic01
cat '/opt/abc.json' | while read line
do
    echo $line | ${kafkaPath}/bin/kafka-console-producer.sh --broker-list ${brokerlist} --topic ${topic}
done
```

