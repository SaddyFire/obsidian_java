## 随机获取MongoDB表中的数据

```java
@Override
...
    public List<Movement> randomMovements(Integer pagesize) {
        /**
         * 创建统计对象, 设置统计参数
         * Aggregation.newAggregation(要查询的表, 查询的函数)
         */
        TypedAggregation<Movement> aggregation = Aggregation.newAggregation(Movement.class, Aggregation.sample(pagesize));
        /**
         * 调用mongoTemplate方法统计
         * mongoTemplate.aggregate(统计对象, 要封装的表)
         */
        AggregationResults<Movement> results = mongoTemplate.aggregate(aggregation, Movement.class);
        return results.getMappedResults();
    }
...
```