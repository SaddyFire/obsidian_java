## 01 需要用到的控制台命令(用于检测)
```json
# 快速查看索引
get /goods/

# 快速查看文档
get /goods/_doc/974401

# 查询全部
get /goods/_search

# 快速删除文档
delete /goods/_doc/562379

# 快速删除索引
delete /goods


GET _search
{
  "query": {
    "match_all": {}
  }
}

# 映射
{
  "mappings":{
    "properties":{
      "id":{
        "type":"integer"
      },
      "title":{
        "type":"text",
        "analyzer": "ik_max_word"
      },
      "price":{
        "type":"double"
      },
      "stock":{
        "type":"double"
      },
      "saleNum":{
        "type":"double"
      },
      "createTime":{
        "type":"date"
      },
      "categoryName":{
        "type":"keyword"
      },
      "brandName":{
        "type":"keyword"
      },
      "spec":{
        "type":"object"
      }
    }
  }
}

```


## 02 实体类
注意: 几个注解的含义及使用

```java

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.annotation.JSONField;
import com.baomidou.mybatisplus.annotation.TableField;
import com.fasterxml.jackson.annotation.JsonFormat;
import jdk.nashorn.internal.ir.annotations.Ignore;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Date;
import java.util.HashMap;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Goods {

    private Integer id;
    private String title;
    private Double price;
    private Integer stock;
    private Integer saleNum;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss" ,timezone = "GMT+8")
    private Date createTime;
    private String categoryName;
    private String brandName;
    @TableField(exist = false)	//排除mysql
    private HashMap spec;
    @TableField("spec")	//映射mysql
    @Ignore	//排除es字段映射
    @JSONField(serialize = false)   //不将该字段序列化为json字符串
    private String specStr;
	
	//重写get方法, 将mysql中的json解析成对象存储至es
    public HashMap getSpec(){
        return JSON.parseObject(specStr, HashMap.class);
    }
}
```

## 03 测试代码实现

##### 3.1 前置后置所需对象
```java
@SpringBootTest
public class MyTest {

    RestHighLevelClient client;

    @Autowired
    private GoodsMapper goodsMapper;

    @BeforeEach
    void t0() {
        // 连接到ES的对象
        client = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://192.168.136.3:9200")));
    }

    @AfterEach
    void t00() throws IOException {
        // 关闭连接
        client.close();
    }

}
```

##### 3.2 新建索引
```java
    //新建索引
    @Test
    void test() throws IOException {
        String mapping = "{\n" +
                "  \"mappings\":{\n" +
                "    \"properties\":{\n" +
                "      \"id\":{\n" +
                "        \"type\":\"integer\"\n" +
                "      },\n" +
                "      \"title\":{\n" +
                "        \"type\":\"text\",\n" +
                "        \"analyzer\": \"ik_max_word\"\n" +
                "      },\n" +
                "      \"price\":{\n" +
                "        \"type\":\"double\"\n" +
                "      },\n" +
                "      \"stock\":{\n" +
                "        \"type\":\"double\"\n" +
                "      },\n" +
                "      \"saleNum\":{\n" +
                "        \"type\":\"double\"\n" +
                "      },\n" +
                "      \"createTime\":{\n" +
                "        \"type\":\"date\"\n" +
                "      },\n" +
                "      \"categoryName\":{\n" +
                "        \"type\":\"keyword\"\n" +
                "      },\n" +
                "      \"brandName\":{\n" +
                "        \"type\":\"keyword\"\n" +
                "      },\n" +
                "      \"spec\":{\n" +
                "        \"type\":\"object\"\n" +
                "      }\n" +
                "    }\n" +
                "  }\n" +
                "}\n";

        //非空判断
        if (client.indices().exists(new GetIndexRequest("goods"), RequestOptions.DEFAULT)) {
            System.out.println("goods表已存在");
            return;
        }
        //
        CreateIndexResponse goods = client.indices().create(new CreateIndexRequest("goods").source(mapping, XContentType.JSON), RequestOptions.DEFAULT);
        System.out.println(goods.isAcknowledged());


    }
```

##### 3.3 添加单条文档
```java
    //添加单条文档
    @Test
    void test3() throws IOException {
        LambdaQueryWrapper<Goods> lqw = new LambdaQueryWrapper<>();
        List<Goods> goodsList = goodsMapper.selectList(lqw);
        //增加单条
        Goods goods = goodsList.get(1);
        IndexResponse goods1 = client.index(new IndexRequest("goods").id(goods.getId().toString()).source(JSON.toJSONString(goods), XContentType.JSON), RequestOptions.DEFAULT);
        String id = goods1.getId();
        System.out.println(id);
    }
```

##### 3.4 批量插入文档
```java
    //批量添加文档
    @Test
    void test03() throws IOException {
        LambdaQueryWrapper<Goods> lqw = new LambdaQueryWrapper<>();
        List<Goods> goodsList = goodsMapper.selectList(lqw);
        //新建批量Request
        BulkRequest request = new BulkRequest();
        goodsList.forEach(good -> {
            request.add(new IndexRequest("goods").id(good.getId().toString()).source(JSON.toJSONString(good),XContentType.JSON));

        });
        //发送请求
        client.bulk(request,RequestOptions.DEFAULT);
    }
```

##### 3.5 term查询
```java
    @Test
    //查询品牌是三星的
    void selectTest() throws IOException {
        //创建request对象
        SearchRequest goodsRequest = new SearchRequest("goods");
        goodsRequest.source().query(QueryBuilders.termQuery("brandName", "三星"));
        SearchResponse search = client.search(goodsRequest, RequestOptions.DEFAULT);
        Arrays.stream(search.getHits().getHits()).forEach(System.out::println);
    }
```

##### 3.6 match查询
```java
    @Test
    //查询title中包含华为手机的
    void selectTest01() throws IOException {
        //创建request对象
        SearchRequest searchRequest = new SearchRequest("goods");
        searchRequest.source().query(QueryBuilders.matchQuery("title", "华为手机"));
        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        Arrays.stream(search.getHits().getHits()).forEach(System.out::println);
    }
```

##### 3.7 完整test代码及需求
```java
import cn.itcast.hotel.mapper.GoodsMapper;
import cn.itcast.hotel.pojo.Goods;
import com.alibaba.fastjson.JSON;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import org.apache.http.HttpHost;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.QueryBuilders;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.Arrays;
import java.util.List;

/**
 * @author SaddyFire
 * @date 2022/1/11
 * @TIME:21:17
 */
@SpringBootTest
public class MyTest {

    RestHighLevelClient client;

    @Autowired
    private GoodsMapper goodsMapper;

    @BeforeEach
    void t0() {
        // 连接到ES的对象
        client = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://192.168.136.3:9200")));
    }

    @AfterEach
    void t00() throws IOException {
        // 关闭连接
        client.close();
    }


    /*
     *  需求:
     *  1. 使用MybatisPlus查询goods表
     *  2. 在ES中创建索引并按照goods表添加映射 (title字段需要分词)
     *  3. 将goods表中的数据导入到ES中
     *  4. 查询品牌是三星的
     *  5. 查询title中包含华为手机的
     */
    //新建索引
    @Test
    void test() throws IOException {
        String mapping = "{\n" +
                "  \"mappings\":{\n" +
                "    \"properties\":{\n" +
                "      \"id\":{\n" +
                "        \"type\":\"integer\"\n" +
                "      },\n" +
                "      \"title\":{\n" +
                "        \"type\":\"text\",\n" +
                "        \"analyzer\": \"ik_max_word\"\n" +
                "      },\n" +
                "      \"price\":{\n" +
                "        \"type\":\"double\"\n" +
                "      },\n" +
                "      \"stock\":{\n" +
                "        \"type\":\"double\"\n" +
                "      },\n" +
                "      \"saleNum\":{\n" +
                "        \"type\":\"double\"\n" +
                "      },\n" +
                "      \"createTime\":{\n" +
                "        \"type\":\"date\"\n" +
                "      },\n" +
                "      \"categoryName\":{\n" +
                "        \"type\":\"keyword\"\n" +
                "      },\n" +
                "      \"brandName\":{\n" +
                "        \"type\":\"keyword\"\n" +
                "      },\n" +
                "      \"spec\":{\n" +
                "        \"type\":\"object\"\n" +
                "      }\n" +
                "    }\n" +
                "  }\n" +
                "}\n";

        //非空判断
        if (client.indices().exists(new GetIndexRequest("goods"), RequestOptions.DEFAULT)) {
            System.out.println("goods表已存在");
            return;
        }
        //
        CreateIndexResponse goods = client.indices().create(new CreateIndexRequest("goods").source(mapping, XContentType.JSON), RequestOptions.DEFAULT);
        System.out.println(goods.isAcknowledged());


    }

    //添加单条文档
    @Test
    void test3() throws IOException {
        LambdaQueryWrapper<Goods> lqw = new LambdaQueryWrapper<>();
        List<Goods> goodsList = goodsMapper.selectList(lqw);
        //增加单条
        Goods goods = goodsList.get(1);
        IndexResponse goods1 = client.index(new IndexRequest("goods").id(goods.getId().toString()).source(JSON.toJSONString(goods), XContentType.JSON), RequestOptions.DEFAULT);
        String id = goods1.getId();
        System.out.println(id);
    }

 /*   //批量插入文档
    @Test
    void test2() throws IOException {
        LambdaQueryWrapper<Goods> lqw = new LambdaQueryWrapper<>();
        List<Goods> goodsList = goodsMapper.selectList(lqw);


        goodsList.forEach(goods -> {
            try {
                client.index(new IndexRequest("goods").id(goods.getId().toString()).source(JSON.toJSONString(goods),XContentType.JSON),RequestOptions.DEFAULT);
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }*/

    //批量添加文档
    @Test
    void test03() throws IOException {
        LambdaQueryWrapper<Goods> lqw = new LambdaQueryWrapper<>();
        List<Goods> goodsList = goodsMapper.selectList(lqw);
        //新建批量Request
        BulkRequest request = new BulkRequest();
        goodsList.forEach(good -> {
            request.add(new IndexRequest("goods").id(good.getId().toString()).source(JSON.toJSONString(good),XContentType.JSON));

        });
        //发送请求
        client.bulk(request,RequestOptions.DEFAULT);
    }

    @Test
    //查询品牌是三星的
    void selectTest() throws IOException {
        //创建request对象
        SearchRequest goodsRequest = new SearchRequest("goods");
        goodsRequest.source().query(QueryBuilders.termQuery("brandName", "三星"));
        SearchResponse search = client.search(goodsRequest, RequestOptions.DEFAULT);
        Arrays.stream(search.getHits().getHits()).forEach(System.out::println);
    }

    @Test
    //查询title中包含华为手机的
    void selectTest01() throws IOException {
        //创建request对象
        SearchRequest searchRequest = new SearchRequest("goods");
        searchRequest.source().query(QueryBuilders.matchQuery("title", "华为手机"));
        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        Arrays.stream(search.getHits().getHits()).forEach(System.out::println);
    }

```





















