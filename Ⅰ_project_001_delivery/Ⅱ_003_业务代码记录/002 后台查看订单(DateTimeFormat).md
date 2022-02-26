## 01 controller

注: 接收**DateTimeFormat** 格式需要使用注解 **@DateTimeFormat** 定义格式

```java
@GetMapping("/page")
@ApiOperation("后台查看订单")
public R page(Integer page, Integer pageSize, Long number,
              @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime beginTime,
              @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime endTime){
    return ordersService.findPage(page,pageSize,number,beginTime,endTime);
}
```

## 02 service

注: DateTimeFormat 格式比较, maytisplus中用 **between()** 方法

<ins>qw.between(Orders::getOrderTime, beginTime, endTime);</ins>

```java
@Override
public R findPage(Integer page, Integer pageSize, Long number, LocalDateTime beginTime, LocalDateTime endTime) {
    LambdaQueryWrapper<Orders> qw = new LambdaQueryWrapper<>();
    if(number != null){
        qw.eq(Orders::getId,number);
    }
    if(beginTime != null && endTime != null ){
        qw.between(Orders::getOrderTime, beginTime, endTime);
    }
    List<Orders> list = this.list(qw);


    List<OrdersDto> ordersDtos = JSON.parseArray(JSON.toJSONString(list), OrdersDto.class);
    ordersDtos = ordersDtos.stream().map(s->{
        if(s.getUserName() == null){
            s.setUserName(s.getUserId().toString());
        }
        LambdaQueryWrapper<OrderDetail> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(OrderDetail::getOrderId,s.getId());
        List<OrderDetail> list1 = orderDetailService.list(wrapper);
        s.setOrderDetails(list1);
        return s;
    }).collect(Collectors.toList());


    IPage<OrdersDto> page1 = new Page<>(page,pageSize);
    page1.setRecords(ordersDtos);
    return R.success(page1);
}
```