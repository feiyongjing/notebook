### ES的java API调用
- 老版的java客户端连接参考：https://www.jianshu.com/p/5cb91ed22956
- 新版的java客户端连接参考：https://www.cnblogs.com/little-lunatic/p/19066460
- 7.x及其以下的版本，spring data有集成支持参考：https://zhuanlan.zhihu.com/p/587439969

### ES7、8集成SpringBoot
1. 引入与ES服务端一致的依赖(别忘了spring-boot-starter-parent和spring-boot-starter-web)
~~~pom
<!-- 新版 Elasticsearch Java Client -->
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
    <version>7.17.29</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
<dependency>
    <groupId>jakarta.json</groupId>
    <artifactId>jakarta.json-api</artifactId>
    <version>2.0.1</version>
</dependency>
~~~

2. 配置文件添加配置
~~~yml
es:
  address: 127.0.0.1
  port: 9200
  scheme: http
  username: admin
  password: admin
  connectTimeout: 5000
  socketTimeout: 30000
~~~

3. 连接配置设置
~~~java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.json.jackson.JacksonJsonpMapper;
import co.elastic.clients.transport.ElasticsearchTransport;
import co.elastic.clients.transport.rest_client.RestClientTransport;
import org.apache.http.HttpHost;
import org.apache.http.auth.AuthScope;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.client.CredentialsProvider;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.elasticsearch.client.RestClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ESClientConfig {

    @Value("${es.address}")
    String address;

    @Value("${es.port}")
    Integer port;

    @Value("${es.scheme}")
    String scheme;

    @Value("${es.username}")
    String username;

    @Value("${es.password}")
    String password;

    @Value("${es.connectTimeout}")
    Integer connectTimeout;

    @Value("${es.socketTimeout}")
    Integer socketTimeout;

    /**
     * 简化版：仅暴露 ElasticsearchClient Bean，底层组件内部创建
     * destroyMethod = "close"：触发链式销毁（Client → Transport → RestClient）
     */
    @Bean(destroyMethod = "close")
    public ElasticsearchClient elasticsearchClient() {
        // 1. 配置认证（可选）
        final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        credentialsProvider.setCredentials(
                AuthScope.ANY,
                new UsernamePasswordCredentials(username, password)
        );

        // 2. 创建 RestClient
        RestClient restClient = RestClient.builder(new HttpHost(address, port, scheme))
                .setHttpClientConfigCallback(httpClientBuilder ->
                        httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider)
                )
                .setRequestConfigCallback(requestConfigBuilder ->
                        requestConfigBuilder
                                .setConnectTimeout(connectTimeout)
                                .setSocketTimeout(socketTimeout)
                )
                .build();

        // 3. 创建 Transport
        ElasticsearchTransport transport = new RestClientTransport(restClient, new JacksonJsonpMapper());

        // 4. 创建 Client（实现类是 Closeable，支持 close() 链式销毁）
        return new ElasticsearchClient(transport);
    }
}
~~~

4. 索引API操作
~~~java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.indices.CreateIndexRequest;
import co.elastic.clients.elasticsearch.indices.CreateIndexResponse;
import co.elastic.clients.elasticsearch.indices.DeleteIndexRequest;
import co.elastic.clients.elasticsearch.indices.DeleteIndexResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.io.IOException;

@Service
@RequiredArgsConstructor
public class IndexOperationsService {

    private final ElasticsearchClient esClient;

    /**
     * # 创建索引
     * PUT /{indexName}  # {indexName} 替换为你要创建的索引名（如 test_index）
     * {
     *   "settings": {
     *     "number_of_shards": "3",  # 分片数（对应 Java 中的 numberOfShards("3")）
     *     "number_of_replicas": "2" # 副本数（对应 Java 中的 numberOfReplicas("2")）
     *   },
     *   "mappings": {
     *     "properties": {
     *       "title": {
     *         "type": "text"  # 文本类型（对应 Java 中的 p.text(t -> t)）
     *       },
     *       "content": {
     *         "type": "text"  # 文本类型
     *       },
     *       "createTime": {
     *         "type": "date"   # 日期类型（对应 Java 中的 p.date(d -> d)）
     *       },
     *       "author": {
     *         "type": "keyword" # 关键字类型（不分词，对应 Java 中的 p.keyword(k -> k)）
     *       }
     *     }
     *   }
     * }
     * @param indexName 索引名称
     * @return 是否成功创建索引
     * @throws IOException
     */
    public boolean createIndex(String indexName) throws IOException {
        CreateIndexResponse response = esClient.indices().create(
                CreateIndexRequest.of(b -> b
                        .index(indexName)
                        .settings(s -> s
                                .numberOfShards("3")
                                .numberOfReplicas("2")
                        )
                        .mappings(m -> m
                                .properties("title", p -> p.text(t -> t))
                                .properties("content", p -> p.text(t -> t))
                                .properties("createTime", p -> p.date(d -> d))
                                .properties("author", p -> p.keyword(k -> k))
                        )
                )
        );

        return response.acknowledged();
    }

    /**
     * # 删除索引
     * DELETE /{indexName}  # {indexName} 替换为要删除的索引名（如 test_article）
     * @param indexName 索引名称
     * @return 是否成功删除索引
     * @throws IOException
     */
    public boolean deleteIndex(String indexName) throws IOException {
        DeleteIndexResponse response = esClient.indices().delete(
                DeleteIndexRequest.of(b -> b.index(indexName))
        );

        return response.acknowledged();
    }
}
~~~

5. 文档API操作（这里只列出了简单的查询操作，复杂的查询操作请查阅官方文档）
~~~java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.*;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.io.IOException;

@Service
@RequiredArgsConstructor
public class DocOperationsService {
    private final ElasticsearchClient esClient;

    private static final ObjectMapper objectMapper = new ObjectMapper();


    /**
     * 添加文档
     * @param indexName 索引名称
     * @param jsonDocument 请求头文档的json字符串
     * @return 随机生成的文档id
     * @throws IOException
     */
    public String addDocument(String indexName, String jsonDocument) throws IOException {
        return addDocument(indexName, null, jsonDocument);
    }


    /**
     * 添加文档
     * 如果不指定文档id是如下post请求
     * POST /{indexName}/_doc/
     * {
     *   "title": "测试标题",
     *   "content": "测试内容",
     *   "createTime": "2026-02-01 12:00:00",
     *   "author": "测试作者"
     * }
     * 有指定文档id是如下put请求
     * PUT /{indexName}/_doc/{id}
     * {
     *   "title": "测试标题",
     *   "content": "测试内容",
     *   "createTime": "2026-02-01 12:00:00",
     *   "author": "测试作者"
     * }
     * @param indexName 索引名称
     * @param id 预期新建的文档id，传null会随机生成文档id
     * @param jsonDocument 请求头文档的json字符串
     * @return 文档id
     * @throws IOException
     */
    public String addDocument(String indexName, String id, String jsonDocument) throws IOException {
        IndexRequest.Builder<JsonNode> requestBuilder = new IndexRequest.Builder<>();
        requestBuilder.index(indexName);
        if (id != null && !id.isEmpty()) {
            requestBuilder.id(id);
        }

        JsonNode jsonNode = objectMapper.readTree(jsonDocument);
        requestBuilder.document(jsonNode);

        IndexResponse response = esClient.index(requestBuilder.build());
        return response.id();
    }


    /**
     * 查询指定id的文档
     * GET /{indexName}/_doc/{id}
     * @param indexName 索引
     * @param id 文档id
     * @return 文档json字符串
     * @throws IOException
     */
    public String getDocument(String indexName, String id) throws IOException {
        GetResponse<JsonNode> response = esClient.get(
                g -> g.index(indexName).id(id), JsonNode.class
        );

        if (response.found()) {
            return response.source().toString();
        } else {
            return null;
        }
    }


    /**
     * 更新文档：注意传入的 JSON 字段合并到原文档（覆盖同名字段，新增不存在的字段，原文档中未被覆盖的字段会被保留，不会因传入的 JSON 缺少该字段而删除）
     * POST /{indexName}/_doc/{id}
     * {
     *   "doc": {                       # 对应 Java 中的 .doc(jsonNode)
     *     "title":"Python新闻",
     *     "content":"关于Java的最新消息",
     *     "createTime":"2023-02-01",
     *     "author":"李四"
     *   }
     * }
     * @param indexName 索引
     * @param id 文档id
     * @param jsonDocument 文档json字符串
     * @return 是否更新成功
     * @throws IOException
     */
    public boolean updateDocument(String indexName, String id, String jsonDocument) throws IOException {
        JsonNode jsonNode = objectMapper.readTree(jsonDocument);
        UpdateResponse<JsonNode> response = esClient.update(
                u -> u.index(indexName).id(id).doc(jsonNode), JsonNode.class
        );

        return response.result().name().equals("Updated");
    }

    /**
     * 删除指定id的文档
     * DELETE /{indexName}/_doc/{id}
     * @param indexName 索引
     * @param id 文档id
     * @return 是否删除成功
     * @throws IOException
     */
    public boolean deleteDocument(String indexName, String id) throws IOException {
        DeleteResponse response = esClient.delete(
                d -> d.index(indexName).id(id)
        );

        return response.result().name().equals("Deleted");
    }

}
~~~











