## 1. 实体类(举例)

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("dish")
public class Dish {
	private Long id; //主键
	private String name; //菜品名称
	@TableField("category_id")
	private Long categoryId; //菜品分类id
	private Integer price; //菜品价格
	private String code; //商品码
	private String image; //图片
	private String description; //描述信息
	private Integer status; //0 停售 1 起售
	private Integer sort; //顺序
	@TableField("create_time")
	private Date createTime; //创建时间
}
```

## 2. PageBean

- totalCount
- List&lt; T&gt;

## 3. Result

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Result {
		private Integer code;		
	private String msg;
	private Object data;
	public Result(Integer code, String msg) {
		this.code = code;
		this.msg = msg;
	}
	public Result(Integer code, Object data) {
		this.code = code;
		this.data = data;
	}
}

```