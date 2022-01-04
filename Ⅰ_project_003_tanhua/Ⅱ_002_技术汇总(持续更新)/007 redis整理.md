## 01 string 使用场景
- 存储验证码
- 存储封禁用户, 可设置封禁时间

```java
    /**
     * 冻结用户
     * @param freezeDto
     */
    public void freeze(FreezeDto freezeDto) {
        Long userId = freezeDto.getUserId();
        Integer freezingTime = freezeDto.getFreezingTime();
        //先查询redis是否有数据
        Object freezeRedis = redisTemplate.opsForValue().get(Constants.FREEZE_USER + userId);

        if(freezeRedis != null){
            throw new BusinessException("异常请求");
        }
        //冻结时间，1为冻结3天，2为冻结7天，3为永久冻结
        int day = 0;
        if (freezingTime == 1) {
            day = 7;
        } else if (freezingTime == 2) {
            day = 1;
        } else if (freezingTime == 3) {
            day = -1;
        }
        String freezeJson = JSON.toJSONString(freezeDto);
        //存储封禁redis
        redisTemplate.opsForValue().set(Constants.FREEZE_USER + userId,freezeJson,day, TimeUnit.DAYS);

    }
```

## 02 hash 使用场景
- 动态点赞 (将点赞结果存储至redis) :

```java
...动态点赞...
String key = Constants.MOVEMENT_LIKE_HASHKEY + movementId;
String haskey = Constants.MOVEMENT_LIKE_HASHKEY + UserHolder.getUserId();
//将动态点赞结果存至redis
redisTemplate.opsForHash().put(key,haskey,"留着以后用");
.............
...取消点赞...
//清除对应redis缓存  
String key = Constants.MOVEMENTS_INTERACT_KEY + movementId;  
String hasKey = Constants.MOVEMENT_LOVE_HASHKEY + UserHolder.getUserId();  
redisTemplate.opsForHash().delete(key,hasKey);
.............

```
	
- 保存上一次访问访问访客列表访问时间 , hset  \=\=>  `Constants.VISITORS_USER`      hkey\=\=>`userId`     hvalue \=\=> `访问时间`

```java
...
//查询设置访问时间
String key = Constants.VISITORS_USER;
String hashKey = String.valueOf(UserHolder.getUserId());
//从redis获取上次查看访问的时间
String value = (String) redisTemplate.opsForHash().get(key, hashKey);
...
//设置当前访问时间
redisTemplate.opsForHash().put(key, hashKey, System.currentTimeMillis());
```

- 小视频点赞
```java
...视频点赞...
//获取当前用户  
Long userId = UserHolder.getUserId();  
//redis上查询是否有点赞数据  
Boolean exist = redisTemplate.opsForHash().hasKey(Constants.VIDEO_LIKE_HASHKEY + videoId, userId);

//redis记录数据  
redisTemplate.opsForHash().put(Constants.VIDEO_LIKE_HASHKEY + videoId,userId,1);
.............
```

## 03 set 使用场景
- 探花喜欢, 用于保存用户双方喜欢关系


```java
...
//更新后存储至redis
redisTemplate.opsForSet().add(Constants.USER_LIKE_KEY + currentUserId, userId.toString());

redisTemplate.opsForSet().remove(Constants.USER_NOT_LIKE_KEY + currentUserId, userId.toString());

//从redis读取, 判断是否双向喜欢, 若双向喜欢则自动加为好友
Boolean islike = redisTemplate.opsForSet().isMember(Constants.USER_LIKE_KEY + userId, currentUserId.toString());
if (islike) {
	friendApi.save(userId,currentUserId);
}
...
```
	
	
#### 3.1 guava中的交集、并集、差集
```java
Set<Integer> set1 = Sets.newHashSet(1, 2, 4, 5, 6, 8);
Set<Integer> set2 = Sets.newHashSet(2, 3, 4, 5, 6, 7, 9);

//合集，并集   [1, 2, 4, 5, 6, 8, 3, 7, 9]
Set<Integer> result1 = Sets.union(set1, set2);
//交集          [2, 4, 5, 6]
Set<Integer> result2 = Sets.intersection(set1, set2);
//差集 1中有而2中没有的  [1, 8]
Set<Integer> result3 = Sets.difference(set1, set2);
//相对差集 1中有2中没有  2中有1中没有的 取出来做结果 [1, 8, 3, 7, 9]
Set<Integer> result4 = Sets.symmetricDifference(set1, set2);

```






