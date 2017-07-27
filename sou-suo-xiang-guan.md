## 搜索相关

`elasticsearch`，基于luence，用于存储索引搜索，支持两种方式，java或者http方式。要求java8.

`logstash`，数据处理，input和output，中间支持filter

`kibana`，展示

### logstash

要导入mysql，需要插件`logstash-input-jdbc`，进入到`logstash/bin`下，运行

`./logstash-plugin install logstash-input-jdbc`命令

下载mysql的jdbc驱动，放到lib下

配置文件jdbc\_test.conf

```
input{
  jdbc{
  jdbc_driver_library => "~/Develop/logstash-5.5.1/lib/third-lib/mysql-connector-java-5.1.43-bin.jar"  #上文下载的mysql的jdbc驱动文件绝对路径
                jdbc_driver_class => "com.mysql.jdbc.Driver" #mysql数据库
                jdbc_connection_string => "jdbc:mysql://localhost:3306/travelid_db"   #格式jdbc:mysql://IP:PORT/数据库名
                jdbc_user => "root"  #数据库用户名
                jdbc_password => "5040309511" #数据库密码
                statement => "SELECT * from masters" #查询语句
  }
}

filter {
        mutate {
                add_field => {
                        "type" => "bdoor-order"  #添加一个type字段，值为bdoor-order
                }
        }
}

output { 
       #stdout { codec => rubydebug } #测试用，将结果打印在终端
       elasticsearch {
        action => create
        index => "travelid_db"
        document_type => "masters"
        document_id => "%{id}"
        hosts => ["http://127.0.0.1:9200"]
        }
}
```

运行

```
in/logstash -f config/jdbc_test.conf
```

```
#http pie 查询
http GET ":9200/travelid_db/masters/_search?q=顾问"

#curl
curl -XGET 'localhost:9200/travelid_db/masters/_search?pretty&q=顾问'
```



