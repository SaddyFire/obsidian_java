> 2dsphere:
>
> `2dsphere`索引支持查询在一个类地球的球面上进行几何计算，以GeoJSON对象或者普通坐标对的方式存储数据。
>
> MongoDB内部支持多种GeoJson对象类型：

## 
```java
//查询附近且获取间距
@Test
public void testNear1() {
    //1. 构造中心点(圆点)
    GeoJsonPoint point = new GeoJsonPoint(116.404, 39.915);
    //2. 构建NearQuery对象
    NearQuery query = NearQuery.near(point, Metrics.KILOMETERS).maxDistance(1, Metrics.KILOMETERS);
    //3. 调用mongoTemplate的geoNear方法查询
    GeoResults<Places> results = mongoTemplate.geoNear(query, Places.class);
    //4. 解析GeoResult对象，获取距离和数据
    for (GeoResult<Places> result : results) {
        Places places = result.getContent();
        double value = result.getDistance().getValue();
        System.out.println(places+"---距离："+value + "km");
    }
}
```

