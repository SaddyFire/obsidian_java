## 01 依赖
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.9.RELEASE</version>
</parent>

<dependencies>
    <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
	
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
	
</dependencies>
```

## 02 application.yml配置文件
```yml
#配置mongo的连接地址
spring:
  data:
    mongodb:
      uri: mongodb://192.168.136.160:27017/testdb
```

<br/>

## 03 启动类(略) & CURD
```java
import cn.itcast.mongo.MongoApplication;
import cn.itcast.mongo.domain.Person;
import org.bson.types.ObjectId;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = MongoApplication.class)
public class MongoTest {

    /**
     * SpringData-mongodb操作
     *    1 配置实体类
     *    2 实体类上配置注解（配置集合和对象间的映射关系）
     *    3 注入MongoTemplate对象
     *    4 调用对象方法，完成数据库操作
     */
    @Autowired
    private MongoTemplate mongoTemplate;

    //保存
    @Test
    public void testSave() {
        for (int i = 0; i < 10; i++) {
            Person person = new Person();
            person.setId(ObjectId.get()); //ObjectId.get()：获取一个唯一主键字符串
            person.setName("张三"+i);
            person.setAddress("北京顺义"+i);
            person.setAge(18+i);
            
            mongoTemplate.save(person);
        }
    }

    //查询-查询所有
    @Test
    public void testFindAll() {
        List<Person> list = mongoTemplate.findAll(Person.class);
        for (Person person : list) {
            System.out.println(person);
        }
    }

    @Test
    public void testFind() {
        //查询年龄小于20的所有人
        Query query = new Query(Criteria.where("age").lt(20)); //查询条件对象
        //查询
        List<Person> list = mongoTemplate.find(query, Person.class);

        for (Person person : list) {
            System.out.println(person);
        }
    }

    /**
     * 分页查询
     */
    @Test
    public void testPage() {
        Criteria criteria = Criteria.where("age").lt(30);
        //1 查询总数
        Query queryCount = new Query(criteria);
        long count = mongoTemplate.count(queryCount, Person.class);
        System.out.println(count);
        //2 查询当前页的数据列表, 查询第二页，每页查询2条
        Query queryLimit = new Query(criteria)
                .limit(2)//设置每页查询条数
                .skip(2) ; //开启查询的条数 （page-1）*size
        List<Person> list = mongoTemplate.find(queryLimit, Person.class);
        for (Person person : list) {
            System.out.println(person);
        }
    }


    /**
     * 更新:
     *    根据id，更新年龄
     */
    @Test
    public void testUpdate() {
        //1 条件
        Query query = Query.query(Criteria.where("id").is("5fe404c26a787e3b50d8d5ad"));
        //2 更新的数据
        Update update = new Update();
        update.set("age", 20);
        mongoTemplate.updateFirst(query, update, Person.class);
    }

    @Test
    public void testRemove() {
        Query query = Query.query(Criteria.where("id").is("5fe404c26a787e3b50d8d5ad"));
        mongoTemplate.remove(query, Person.class);
    }
}

```



