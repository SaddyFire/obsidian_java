数据从浏览器进入到Contorller有几种格式
3种	application/json(请求体)	application/x-www-form-urlencoded(可以在请求体/可以在请求行)
		multipart/form-data(只能是在请求体中)
		数据从浏览器进入到Controller中可以是请求体 也可以是在请求行


## 01 请求参数为Query简单类型
1. 如果只有一个 我们直接接受即可 参数名要和传递的参数名字一致 如果非要不一致的可以在参数前添加@RequestParam(“userId”)
public ResponseEntity aaa(`@RequestParam("userId")` Long userId)

2. 如果有多个
	1. 我们用多个参数接收,每个参数前加上@RequestParam注解
	public ResponseEntity aaa(`@RequestParam("userId")` Long userId ,`@RequestParam("name")`String name)
	
	**注意:\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=** 
	如果用 `@RequestPram`注解要加映射的字段名
	**注意:\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=** 
	
	2. 可以用Map集合接收用 Map接收需要加上@RequestParam(required = false)
	public ResponseEntity aaa(`@RequestParam(required = false)` Map map)
	
	3. 用实体类接收 但实体类的参数类型和参数名要和传递的参数一致 SpringBoot会帮我们自动填充到实体中@ModelAttribute可以不写
	public ResponseEntity aaa(`User` user)
	public ResponseEntity aaa(`@ModelAttribute` User user)
	
	4. @RequestParam注解还可以设置默认值 `@RequestParam`(defaultValue = “2000”)


## 02 请求参数为Body
前端传递过来的为json,我们可以用Map 接收
**注意:\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=** 
当请求方式为get时, 无法用`@RequestBody`接收
**注意:\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=** 
spring默认会把数据写到Map中 我们在从map中获取即可
public ResponseEntity aaa(`@RequsetBody` Map map)

## 03 请求参数在路径中
@GetMapping("/{id}/love") 
public ResponseEntity tanhuaLove(`@PathVariable("id") `Long likeUserId)

我们就如上接收即可 用@PathVariable(“id”) 如果参数和路径的相同 则可以省略注解的参数

## 04 rest风格
get
post 新增
put 修改
delete 删除
