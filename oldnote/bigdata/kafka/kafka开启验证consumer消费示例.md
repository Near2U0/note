



> kafka开启auth时需要，kafka的consumer去消费

```json
#cat 


[root@kafka config]# 
[root@kafka config]# cat consumer_test.properties


group.id=test-consumer-group


sasl.mechanisms=PLAIN

security.protocol=SASL_PLAINTEXT

sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="gass_cons@new_portalsite_audit" password="gassYFszzf12#$";

####################注意后面一行有分号############
[root@kafka config]# 


#客户端消费，需要指定上面的配置文件
./kafka-console-consumer.sh --bootstrap-server kafka-server:30001  --topic new_portalsite_audit  --from-beginning   --consumer.config ../config/consumer_test_cys.properties 

```

> 注意上面的`sasl.jaas.config`后面要加分号。

