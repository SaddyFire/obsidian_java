> 2dsphere:
> `2dsphere`索引支持查询在一个类地球的球面上进行几何计算，以GeoJSON对象或者普通坐标对的方式存储数据。
> MongoDB内部支持多种GeoJson对象类型：

## 01 查询附近且获取间距

```java
...
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
...
```


## 02 查询附近的人
```java
...
@Override  
public List<UserLocation> searchNearUser(Long userId, String gender, String distance) {  
	//获取当前用户地理信息  
	Query query = Query.query(Criteria.where("userId").is(userId));  
	UserLocation userLocation = mongoTemplate.findOne(query, UserLocation.class);  
	if(userLocation == null){  
	return null;  
	}  
	//构造圆心  
	GeoJsonPoint point = userLocation.getLocation();  
	//构造半径  
	Distance userDistance = new Distance(Double.parseDouble(distance) / 1000, Metrics.KILOMETERS);  
	//绘制原形  
	Circle circle = new Circle(point, userDistance);  
	//查询, withSphere()方法  
	Query locationQuery = Query.query(Criteria.where("location").withinSphere(circle));  
	List<UserLocation> userLocationList = mongoTemplate.find(locationQuery, UserLocation.class);  
 return userLocationList;  
}
...
```


## 03 地理位置信息对应的MongoDB实体类

**注意:** <font color=#bbb529>@CompoundIndex</font>(name = <font color=#6a8759>"location_index"</font>, def = <font color=#6a8759>"{'location': '2dsphere'}"</font>)  

```java
  
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
@Builder  
@Document(collection = "user_location")  
@CompoundIndex(name = "location_index", def = "{'location': '2dsphere'}")  
public class UserLocation implements java.io.Serializable {  
  
 private static final long serialVersionUID = 4508868382007529970L;  
  
 @Id  
 private ObjectId id;  
 @Indexed  
 private Long userId; //用户id  
 private GeoJsonPoint location; //x:经度 y:纬度  
 private String address; //位置描述  
 private Long created; //创建时间  
 private Long updated; //更新时间  
 private Long lastUpdated; //上次更新时间  
}
```







