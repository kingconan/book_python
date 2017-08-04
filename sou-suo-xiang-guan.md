## 搜索相关

`elasticsearch`，基于luence，用于存储索引搜索，支持两种方式，java或者http方式。要求java8.

`logstash`，数据处理，input和output，中间支持filter

`kibana`，展示

### 启动`elasticsearch`

`elasticsearch`安装目录运行 `bin/elasticsearch`

访问 `http://localhost:9200/`可以看到一些信息

##### 通过url进行搜索

```
//travelid_db下的users表，id为1的数据
http://localhost:9200/travelid_db/users/1

//搜索金刚，木有指定column
http://localhost:9200/travelid_db/users/_search?q=%E9%87%91%E5%88%9A

//搜索金刚指定column
http://localhost:9200/travelid_db/users/_search?q=nickname:%E9%87%91%E5%88%9A

//DSL查询，可以构建复杂查询
```

##### 使用php搜索

```php
public function searchHotel(Request $request){
    $q = $request->input("q");
    $city = $request->input("city");
    $client = new Client();
    $request_para = [
        "query" => [
            "bool" =>[
                "must" => [
                    [
                        "match" => [ "name_cn"=>$q ]
                    ],
                    [
                        "match" => [ "city_name_cn"=>$city ]
                    ]
                ]
            ]
        ],
        "highlight"=> [
//                "pre_tags"=> ["<b>"],
//                "post_tags"=> ["</b>"],
            "fields" => ["name_cn" => ["type"=> "plain"]]
        ]
    ];
    $response = $client->request("GET","http://localhost:9200/hq_hotel/hotel/_search",
        [
            "json" => $request_para
        ]);
    if($response->getStatusCode() == 200){
        $json_string =  $response->getBody()->getContents();


        return response()->json(
            [
                'ok' => 0,
                'msg' => 'ok',
                'obj' => \GuzzleHttp\json_decode($json_string)
            ]);
    }
}
```

##### 使用python导入数据

```py
# coding=utf-8

from bs4 import BeautifulSoup
import json
import datetime
import os.path

from elasticsearch import Elasticsearch
import elasticsearch.helpers

def es_hotels():
    print "start hotels"
    es = Elasticsearch()

    set_mapping(es)

    with open(city_json) as f:
        city_arr = json.load(f)
        f.close()
        count = 0
        for city in city_arr:
            hotel_json = hotelJsonFolder + "hotellist_cityId%s.json" % city["id"]
            if not os.path.isfile(hotel_json):
                continue

            # insert one json file

            count += 1

            with open(hotel_json) as hotel_file:
                hotel_arr = json.load(hotel_file)
                hotel_file.close()

                actions = [
                    {
                        "_op_type": "index",
                        "_index": "hq_hotel",
                        "_type": "hotel",
                        "_source": d
                    }
                    for d in hotel_arr
                ]
                elasticsearch.helpers.bulk(es, actions)
                
                #如果使用并行方法，记住parallel_bulk返回的是generator，所以要调用下
                #from collections import deque
                #deque(elasticsearch.helpers.parallel_bulk(es, actions), maxlen=0)
                #

            if count % 100 == 0:
                print count
                now()

    print "end"
    now()


# 设置索引规则，bulk之前调用一下 
def set_mapping(es, index_name="hq_hotel", doc_type_name="hotel"):
    my_mapping = '''
    {
      "mappings":{
        "hotel":{
          "properties":{
            "city_name_cn":{
                "type": "string",
                "index": "not_analyzed"
            },
            "country_name_cn": {
                "type": "string",
                "index": "not_analyzed"
            },
            "name_cn":{
                "type": "string",
                "index": "analyzed"
            },
            "name_en":{
                "type": "string",
                "index": "analyzed"
            }
          }
        }
      }
    }'''
    if es.indices.exists(index_name):
        es.indices.delete(index=index_name)
        print "delete exits index " + index_name

    print "create index"
    es.indices.create(index=index_name, body=my_mapping)
```

##### ubuntu上部署

安装java8或者open-jdk8

设置源，参见[https://www.elastic.co/guide/en/elasticsearch/reference/1.7/setup-repositories.html](https://www.elastic.co/guide/en/elasticsearch/reference/1.7/setup-repositories.html)

设置配置文件，把`network.host: 0.0.0.0` 内外网均可以可以访问

`注意，服务重启是需要时间，所以立刻访问会出现 connection refused`

直接下载部署时，要更改文件属性 `sudo chmod -R 777 es_folder`

否则报奇怪的权限错误

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
bin/logstash -f config/jdbc_test.conf
```

```
#http pie 查询
http GET ":9200/travelid_db/masters/_search?q=顾问"

#curl
curl -XGET 'localhost:9200/travelid_db/masters/_search?pretty&q=顾问'
```



