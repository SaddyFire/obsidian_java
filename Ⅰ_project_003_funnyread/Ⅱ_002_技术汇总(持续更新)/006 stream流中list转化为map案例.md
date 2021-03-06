##### 知识点汇总:
1. 首先api层拿到数据后先进行**非空判断**
2. 使用stream流 **方法引用** 提取集合中的某个元素
3. 使用stream流collect.toMap 将**list映射为map**
4. stream中的map映射要注意总结何时**return**

*未实现过滤版*
```java
 @Override
 ...
    public PageResult friendsMovement(Integer page, Integer pagesize) {
        Long userId = UserHolder.getUserId();
        List<Movement> friendsMovements = movementApi.getFriendsMovement(page,pagesize,userId);
        //1. List<Movement>非空判断
        if(friendsMovements.size() == 0){
            return new PageResult(page,pagesize,0L,friendsMovements);
        }
		
        //2. 提取userId(方法引用)
        List<Long> userIds = friendsMovements.stream().map(Movement::getUserId).collect(Collectors.toList());
        List<UserInfo> userInfos = userInfoApi.findByCondition(userIds, null);
		
		//3. list 映射为 map
        Map<Long, UserInfo> userInfoMap = userInfos.stream().collect(Collectors.toMap(userInfo -> userId, Function.identity()));
		//(或者采用方法引用)
		Map<Long, UserInfo> userInfoMap = userInfoList.stream().collect(Collectors.toMap(UserInfo::getId, Function.identity()));
        List<MovementsVo> list = new ArrayList<>();
		
        //4. 将查询到的结果映射后封装
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

*增加过滤版*
```java
...
	List<Visitors> visitorsList = visitorsApi.findVisitorRecord(currentUserId,date);
	//非空判断
	if(CollUtil.isEmpty(visitorsList)){
		return new ArrayList<>();
	}
	//抽取userId
	List<Long> userIdList = visitorsList.stream()
			.filter(visitor -> visitor.getUserId() != null)
			.map(Visitors::getUserId)
			.collect(Collectors.toList());
	//拿取userInfo
	List<UserInfo> userInfoList = userInfoApi.findByCondition(userIdList, null);
	Map<Long, UserInfo> userInfoMap = userInfoList.stream().collect(Collectors.toMap(UserInfo::getId, Function.identity()));

	//封装VistorsVo
	List<VisitorsVo> visitorsVoList = visitorsList.stream().filter(visitors -> userInfoMap.containsKey(visitors.getUserId())).map(visitors -> {
		return VisitorsVo.init(userInfoMap.get(visitors.getUserId()), visitors);
	}).collect(Collectors.toList());

	return visitorsVoList;
...
```


##### 非空**Lambda表达式** -> 非空判断 , 单变量的方法引用 -> 初始化类型

```java
...
List<NearUserVo> nearUserVoList = userInfoList.stream()
											  .filter(Objects::nonNull)
											  .map(NearUserVo::init)
											  .collect(Collectors.toList());
...
```


##### stream流取交集








