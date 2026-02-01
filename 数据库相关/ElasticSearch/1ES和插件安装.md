### ElasticSearch安装
到github：https://github.com/elastic/elasticsearch/releases 下载对应版本安装

~~~
常见目录如下
home	Elasticsearch 主目录或 $ES_HOME	解压压缩包创建的目录	
bin	    二进制脚本，包括启动节点的 elasticsearch 和安装插件的 elasticsearch-plugin	          $ES_HOME/bin	
jdk     启动需要的jdk，一般情况下如果你已经安装了jdk可以修改一下启动脚本改成使用这个jdk启动
conf	配置文件，包括 elasticsearch.yml（可以配置修改数据文件目录和日志文件目录）	            $ES_HOME/config	 
data	分配在节点上的每个索引和分片的数据文件位置。可以有多个位置。	                        $ES_HOME/data	   path.data
logs	日志文件位置	                                                                    $ES_HOME/logs	   path.logs
plugins	插件文件位置。每个插件会包含在一个子目录中。	                                       $ES_HOME/plugin


# windows启动bin下的bat文件，然后访问 Elasticsearch 健康检查接口
curl http://127.0.0.1:9200/_cluster/health?pretty

# 返回json示例
{
  "cluster_name" : "elasticsearch",
  "status" : "green",  // green 表示健康，yellow 也正常（单机无副本）
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
~~~

### head插件安装
到github：https://github.com/mobz/elasticsearch-head 下载项目启动

~~~
# 下载启动
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start
# 浏览器打开 http://localhost:9100/


# 连接出现跨越问题可以在elasticsearch.yml中添加如下配置然后重新启动ES
http.cors.enabled: true
http.cors.allow-origin: "*"
~~~

### Kibana安装
到github：https://github.com/elastic/kibana/releases 下载与ES一致的版本进行安装
~~~
# 设置汉化：找到和bin目录同级的config目录，进入后编辑kibana.yaml配置文件，使用如下设置
i18n.locale: "zh-CN"

# 设置ES连接配置，默认不写是连接localhost:9200
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/http_ca.crt" ]
elasticsearch.username: "kibana_system"
elasticsearch.password: "password"
elasticsearch.hosts: ["https://127.0.0.1:9200"]

# windows启动bin下的bat文件，浏览器打开 http://localhost:5601/
~~~