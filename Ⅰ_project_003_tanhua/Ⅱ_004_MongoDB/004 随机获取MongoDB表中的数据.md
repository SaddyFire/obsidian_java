## 随机获取MongoDB表中的数据

```java
@Override
...
    public List<Movement> randomMovements(Integer pagesize) {
        /**
         * 创建统计对象, 设置统计参数
         * Aggregation.newAggregation(要查询的表, 查询的函数)
         */
        TypedAggregation<Movement> aggregation = Aggregation.newAggregation(Movement.class,
		Aggregation.sample(pagesize));
        /**
         * 调用mongoTemplate方法统计
         * mongoTemplate.aggregate(统计对象, 要封装的表)
         */
        AggregationResults<Movement> results = mongoTemplate.aggregate(aggregation, Movement.class);
        return results.getMappedResults();
    }
...
```


**加强版**

```java
...
//定义标准
Criteria criteria = Criteria.where("toUserId").nin(likeUserIdList);

TypedAggregation aggregation = Aggregation.newAggregation(RecommendUser.class,  //确定查询类型
														Aggregation.sample(displayCount),   //定义查询数量
														Aggregation.match(criteria));       //匹配标准

//调用 mongoTemplate.aggregate() 进行集成查询
AggregationResults<RecommendUser> results = mongoTemplate.aggregate(aggregation, RecommendUser.class);
//取值
List<RecommendUser> recommendUserList = results.getMappedResults();
...
```