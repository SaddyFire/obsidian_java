## 1\. springMVC的底层原理

![[b3e6ef8496b743b78014b82e108ff86b.png]]

## 1. Controller类

```java
@RestController
@RequestMapping("/books")
public class BookController {
    @Autowired
    private BookService bookService;

    @PostMapping
    public Result add(@RequestBody Book book){
        Boolean flag = bookService.add(book);
        //int i = 1/0;      //测试错误
        Result result = new Result(flag, null, flag == true?"添加成功":"添加失败");
        return result;
    }
    @DeleteMapping
    public Result delete(@RequestBody Integer[] ids){
        Boolean flag = bookService.delete(ids);

        Result result = new Result(flag,null,flag == true?"删除成功":"删除失败");
        return result;
    }
    @PutMapping
    public Result modify(@RequestBody Book book){
        Boolean flag = bookService.modify(book);
        Result result = new Result(flag,null, flag==true?"修改成功":"修改失败");
        return result;
    }
    @GetMapping("/{currentPage}/{pageSize}")
    public Result findByPageAndCondition(@PathVariable Integer currentPage,@PathVariable Integer pageSize,@RequestBody Book book){
        PageBean<Book> pageBean = bookService.findByPageAndCondition(currentPage,pageSize,book);
        Result result = new Result(pageBean.getTotleCount() > 0, null,pageBean.getTotleCount() == 0?"暂无数据":"");
        return result;
    }

}
```