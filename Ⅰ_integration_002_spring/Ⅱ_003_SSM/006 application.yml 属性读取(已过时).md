## 1. application.yml类(详细见mybatis高级-P10)

```yml
spring:
  profiles:
    active: dev

---
spring:
  profiles: dev

server:
  port: 2323
  servlet:
    context-path: /aa

bookdata:
  book_id: 1
  book_name: java入门到劝退
  book_type: 言情
  book_price: 233

---
spring:
  profiles: test
server:
  port: 3333
```

## 2. 属性读取

1. 通过@Value("${bookdata.book_id}")
2. 通过装配Environment => 调用environment.getProperty()
3. 通过对象自动装配

```java
@RestController
@RequestMapping("/books")
@EnableConfigurationProperties(Book.class)
public class BookController {

    @Value("${bookdata.book_id}")
    private String bookId;

    @Autowired
    private Environment environment;

    @Autowired
    private Book book;

    @GetMapping("/{id}")
    public String findById(@PathVariable Integer id){
        System.out.println("第一种方式: "+ bookId);

        System.out.println("第二种方式: "+environment.getProperty("bookdata.book_name"));

        System.out.println("第三种方式: "+book);

        return "welcome to springboot: " + id;
    }
}
```