##### 知识点汇总:
1. 首先api层拿到数据后先进行**非空判断**
2. 使用stream流 **方法引用** 提取集合中的某个元素
3. 使用stream流collect.toMap 将**list映射为map**
4. stream中的map映射要注意总结何时**return**

```java
 @Override
 ...
    public PageResult friendsMovement(Integer page, Integer pagesize) {
        Long userId = UserHolder.getUserId();
        List<Movement> friendsMovements = movementApi.getFriendsMovement(page,pagesize,userId);
        //List<Movement>非空判断
        if(friendsMovements.size() == 0){
            return new PageResult(page,pagesize,0L,friendsMovements);
        }
        //提取userId(方法引用)
        List<Long> userIds = friendsMovements.stream().map(Movement::getUserId).collect(Collectors.toList());
        List<UserInfo> userInfos = userInfoApi.findByCondition(userIds, null);
		//list 映射为 map
        Map<Long, UserInfo> userInfoMap = userInfos.stream().collect(Collectors.toMap(userInfo -> userId, Function.identity()));
        List<MovementsVo> list = new ArrayList<>();
        //将查询到的结果映射后封装
        friendsMovements.stream().map(movement -> {
            MovementsVo init = MovementsVo.init(userInfoMap.get(movement.getUserId()), movement);
            //此处需return
			return list.add(init);
        });

        PageResult pageResult = new PageResult(page,pagesize,0L,list);

        return pageResult;
    }
...
```