## 1. 装配 **HttpServletRequest (具体见06test)**

```java
//1.装配 HttpServletRequest
@Autowired
private HttpServletRequest httpServletRequest;
```

- 补充:  通过HttpServletRequest对象可以拿到URI和URD
    

```java
//URI获取
String requestURI = httpServletRequest.getRequestURI();

//URL获取
StringBuffer requestURL = httpServletRequest.getRequestURL();
```

## 2. 获取session对象 - 设置参数 - 使用参数

```java
//2. 获取session对象
HttpSession session = httpServletRequest.getSession();

//3. 设置参数
session.setAttribute(number,code);

//4. 获取参数
Integer code = (Integer) session.getAttribute(phone.getPhone());
```

## 3. 举例

```java
/**
 * <p>
  * 前端控制器
  * </p>
  *
 * @author SaddyFire
 * @since 2021-11-19
 */@RestController
@RequestMapping("/user")
public class UserController {
	@Autowired
	private UserService userService;
	@Autowired
	private HttpServletRequest httpServletRequest;
	@PostMapping("/sendMsg")
	public Result register(@RequestBody Phone phone){
		HttpSession session = httpServletRequest.getSession();
		String number = phone.getPhone();
		if(number == "" || number == null){
			return new Result(0, "请输入正确的手机号");
		}
		Integer code =  userService.register(number);
		session.setAttribute(number,code);
		return new Result(1,"发送成功,请注意查收");
	}
	@PostMapping("/login")
	public Result login(@RequestBody Phone phone){
		if(phone == null){
		return new Result(0, "非法参数");
	}
	
	HttpSession session = httpServletRequest.getSession();
	Integer code = (Integer) session.getAttribute(phone.getPhone());
	System.out.println(code);
	System.out.println(phone.getCode());
	
	if(!phone.getCode().equals(code)){
		return new Result(0, "验证码错误");
	}
	
	//此时已经通过验证, 清除session
	session.setAttribute(phone.getPhone(),null);
	//执行登陆操作, 若没有数据则注册一个
	return userService.login(phone.getPhone());
	}
}
```