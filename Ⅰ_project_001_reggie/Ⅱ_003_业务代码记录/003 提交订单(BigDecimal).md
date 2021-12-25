## 01 controller

```
@PostMapping("/submit")
@ApiOperation("订单提交")
public R submit(@RequestBody Orders orders){
  return ordersService.submit(orders);
}
```

## 02 service

注:

- **BigDecimal** 的使用:
- **AtomicInteger** 原子类&原子性的使用:

amount.addAndGet(s.getAmount().multiply(new BigDecimal(s.getNumber())).intValue()); //累加所有商品的金额

AtomicInteger amount = new AtomicInteger(**int initialValue**)

```java
@Override
public R submit(Orders orders) {
	//添加订单表
	//添加订单明细表
	//删除购物车
	//获取用户身份
	Long currentId = BaseContext.getCurrentId();
	//获取购物车
	LambdaQueryWrapper<ShoppingCart> lqw = new LambdaQueryWrapper<>();
	lqw.eq(ShoppingCart::getUserId,currentId);
	List<ShoppingCart> shoppingCarts = shoppingCartMapper.selectList(lqw);
	if(shoppingCarts == null || shoppingCarts.size() == 0){
		return R.error("购物车不能为空");
	}
	//根据userid 获取username
	LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
	wrapper.eq(User::getPhone,currentId);
	User user = userMapper.selectOne(wrapper);
	String userName = user.getName();
	orders.setUserName(userName); //设置用户名
	orders.setUserId(currentId); //设置用户id
	long orderId = IdWorker.getId(); //生成订单号
	orders.setId(orderId);
	orders.setStatus(2);
	orders.setOrderTime(LocalDateTime.now());
	orders.setCheckoutTime(LocalDateTime.now());
	//根据用户id查询地址簿
	AddressBook addressBook = addressBookMapper.selectById(orders.getAddressBookId());
	orders.setConsignee(addressBook.getConsignee());
	//设置地址簿
	orders.setAddress((addressBook.getProvinceName() == null ? "" : addressBook.getProvinceName())
	+ (addressBook.getCityName() == null ? "" : addressBook.getCityName())
	+ (addressBook.getDistrictName() == null ? "" : addressBook.getDistrictName())
	+ (addressBook.getDetail() == null ? "" : addressBook.getDetail()));
	orders.setPhone(addressBook.getPhone());
	AtomicInteger amount = new AtomicInteger(0);
	//复核订单金额
	//查询购物车 -> 添加至订单明细
	List<OrderDetail> orderDetails = shoppingCarts.stream().map(s->{
		OrderDetail orderDetail = new OrderDetail();
		orderDetail.setDishId(s.getDishId());
		orderDetail.setName(s.getName());
		orderDetail.setOrderId(orderId);
		orderDetail.setDishFlavor(s.getDishFlavor());
		orderDetail.setNumber(s.getNumber());
		orderDetail.setSetmealId(s.getSetmealId());
		orderDetail.setImage(s.getImage());
		orderDetail.setAmount(s.getAmount());

		amount.addAndGet(s.getAmount().multiply(new BigDecimal(s.getNumber())).intValue()); //累加所有商品的金额

		return orderDetail;
	}).collect(Collectors.toList());
	//存储数据
	orderDetailService.saveBatch(orderDetails);
	//订单设置总金额
	orders.setAmount(new BigDecimal(amount.get()));
	this.save(orders); //存储订单列表
	//清空购物车
	shoppingCartMapper.delete(lqw);
	return R.success("下单成功");
}
```