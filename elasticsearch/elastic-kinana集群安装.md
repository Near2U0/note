[toc]



# 安装ES



```shell
# 1.修改系统的ulimit , vm


# 更改es 安装包权限
chown -R elastic:elastic elasticsearch-6.8.5


# yml配置
node, cluster, data, logs,

discovery.zen.minimum_master_nodes: 2 （防止脑裂）

# jvm
# vim jvm.options 
-Xms20g
-Xmx20g

```





# 安装kibana





# 基于logstash的ES集群间的数据迁移

```json

		input {
    		elasticsearch {
        		hosts => ["http://172.16.0.39:9200"]
        		index => "*"
        		docinfo => true
	    	}
		}
		output {
    		elasticsearch {
        		hosts => ["http://172.16.0.20:9200"]
        		index => "%{[@metadata][_index]}"
    		}
		}


input {
     elasticsearch {
       hosts => "http://xxxxxxxxx:9200"
       user  => "elastic"
       index => "*"
       password => "xxxxxx"
       docinfo => true
     }
   }
output {
      elasticsearch {
        hosts => "http://xxxxxxx.elasticsearch.aliyuncs.com:9200"
        user => "elastic"
        password => "xxxxxx"
        index => "%{[@metadata][_index]}"
        document_type => "%{[@metadata][_type]}"
        document_id => "%{[@metadata][_id]}"
  }
}








input {
    #105
     elasticsearch {
       hosts => ["http://192.168.190.105:9200"]
       user  => "elastic"
       index => "logstash-secdev-2021.09.14"
       password => "****"
       docinfo => true
     }
   }
output {
      #cluster
      elasticsearch {
        hosts => ["http://10.129.8.187:9200", "http://10.129.8.188:9200", "http://10.129.8.189:9200"]
        user => "elastic"
        password => "***"
        index => "%{[@metadata][_index]}"
        document_type => "%{[@metadata][_type]}"
        document_id => "%{[@metadata][_id]}"
  }
}




    elasticsearch{
     user => 'elastic'
     password => 'j[j8o2HKaE+=:^%W'
     index => "logstash-secaudit-%{date}"
     hosts => ["http://192.168.190.105:9200"]
    }



```

