- ## 追加druid依赖
    

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.12</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.1</version>
</dependency>
```

# 1\. application.yml类

```yml
#指定特性配置
spring:
  profiles:
    active: pro
#指定端口
server:
  port: 2323

  #配置数据源的四个信息 -- 创建DataSource并添加到容器
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/mybatis?serverTimezone=TUC
    username: root
    password: 2323
    type: com.alibaba.druid.pool.DruidDataSource
```

# 2\. dao接口

- MyBatisPlus(继承BaseMapper&lt; T&gt;)

```java
@Mapper
public interface UserDao extends BaseMapper<User> {

}
```

- 原始做法(添加@Mapper注解)

```java
//生成代理对象
@Mapper
public interface BookDao {
    /**
     * 增加
     * @param book
     * @return
     */
    @Insert("insert into tbl_book VALUES (null,#{type},#{name},#{description})")
    Integer add(Book book);

    /**
     * 根据id数组删除
     * @param ids
     * @return
     */
    Integer delete(Integer[] ids);

    /**
     * 修改
     * @param book
     * @return
     */
    Integer modify(Book book);

    /**
     * 根据条件查询
     * @param begin
     * @param count
     * @return
     */
    List<Book> findByPageAndCondition(@Param("begin") Integer begin, @Param("count")  Integer count, @Param("book")  Book book);

    /**
     * 根据条件查询数量
     * @param begin
     * @param count
     * @param book
     * @return
     */
    int findCountByPageAndCondition(@Param("begin") Integer begin,@Param("count")  Integer count,@Param("book")  Book book);
}
```

# ~~3\. sql难点语句(已过时)~~

```xml
<delete id="delete">
    DELETE FROM tbl_book
    <where>
        id in
        <foreach collection="ids" item="id" open="(" close=")" separator=",">
            #{id}
        </foreach>
    </where>

</delete>

<update id="modify">
    UPDATE tbl_book
    <set>
        <if test="type != null and type != ''">
            , `type` = #{type}
        </if>
        <if test="name != null and name != ''">
            , `name` = #{name}
        </if>
        <if test="description != null and description != ''">
            ,  description = #{description}
        </if>
    </set>
    WHERE id = #{id}
</update>


<select id="findByPageAndCondition" resultType="com.itheima.pojo.Book">
    SELECT * from tbl_book
    <where>
        <if test="book.type != null and book.type != ''">
            and type  like #{book.type}
        </if>
        <if test="book.name != null and book.name != ''">
            and name  like #{book.name}
        </if>
        <if test="book.description != null and book.description != ''">
            and description  like #{book.description}
        </if>
        limit #{begin}, #{count}
    </where>
</select>



<select id="findCountByPageAndCondition" resultType="integer">
    SELECT count(*) from tbl_book
    <where>
        <if test="book.type != null and book.type != ''">
            and type  like #{book.type}
        </if>
        <if test="book.name != null and book.name != ''">
            and name  like #{book.name}
        </if>
        <if test="book.description != null and book.description != ''">
            and description  like #{book.description}
        </if>
        limit #{begin}, #{count}
    </where>
</select>
```