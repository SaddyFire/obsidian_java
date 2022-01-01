## 01 string 使用场景

## 02 hash 使用场景

## 03 set 使用场景
探花喜欢, 用于保存用户双方喜欢关系

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







