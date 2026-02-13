# SpringBoot接入MongoDB
1. 引入对应版本的依赖
~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
~~~

2. 配置文件添加配置
~~~yml
spring:
  data:
    # MongoDB数据库
    mongodb:
      host: 127.0.0.1                 # 连接地址
      port: 27017                     # 端口
      database: test                  # 连接数据库
      username: "mongo"                 # 用户名
      password: "123456"                # 密码
      authentication-database: admin  # MongoDB 创建用户时默认会关联到某个数据库（通常是 admin 库，超级用户必须创建在 admin 库），连接时必须指定用户关联的库
~~~

3. 使用MongoTemplate 
~~~java
import com.mongodb.client.result.DeleteResult;
import com.mongodb.client.result.UpdateResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService {
    @Autowired
    private MongoTemplate mongoTemplate;

    private static String collectionName = "users";


    public void createCollection() {
        mongoTemplate.createCollection(collectionName);
    }

    public void dropCollection() {
        mongoTemplate.dropCollection(collectionName);
    }

    public boolean collectionExists() {
        return mongoTemplate.collectionExists(collectionName);
    }


    public void addUser() {
        User user = new User();
        user.setName("张三");
        user.setAge(18);
        User user1 = mongoTemplate.insert(user, collectionName);
    }

    public void deleteUser() {
        Criteria criteria = Criteria.where("name").is("张三").and("age").is(18);
        Query query = Query.query(criteria);
        DeleteResult remove = mongoTemplate.remove(query, collectionName);
    }


    public void updateUser() {
        Criteria criteria = new Criteria();
        criteria.orOperator(Criteria.where("name").is("张三"), Criteria.where("age").is(18));
        Query query = Query.query(criteria);
        Update update1 = Update.update("age", 10);

        // updateMulti方法更新与给定查询匹配的所有数据记录
        // updateFirst方法更新与查询匹配的第一条记录
        UpdateResult result = mongoTemplate.updateMulti(query, update1, collectionName);
    }



    public void findUser() {
        Criteria criteria = Criteria.where("name").is("张三");
        Query query = Query.query(criteria);
        User one = mongoTemplate.findOne(query, User.class);
    }


    public void pageUser() {
        Query query = Query.query(new Criteria()).skip(0).limit(10);
        List<User> users = mongoTemplate.find(query, User.class);
    }

    static class User implements Serializable {
        private String name;
        private Integer age;

        public User() {
        }

        public User(String name, Integer age) {
            this.name=name;
            this.age=age;
        }

        public String getName() {
            return name;
        }

        public Integer getAge() {
            return age;
        }

        public void setName(String name) {
            this.name = name;
        }

        public void setAge(Integer age) {
            this.age = age;
        }
    }
}
~~~

