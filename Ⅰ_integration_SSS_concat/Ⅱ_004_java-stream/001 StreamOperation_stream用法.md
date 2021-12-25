1. filter	-> 过滤
2. map	-> 接收自己选择的内容
3. flatMap	-> 接收新的流对象
4. peek	-> 分支循环, 不改变原有流
5. concat	-> 合并
6. **sorted**	-> 排序	-> 用法: .sorted(Comparator.comparing(Sku::getTotalPrice).reversed())
7. distinct	-> 去重
8. skip	-> 跳过
9. limit	-> 截取
10. allMatch, anyMatch, noneMatch	-> 遍历流并匹配对应条件, 返回匹配对应的**boolean**结果
11. findFirst, findAny	-> 查找第一个, 查找任意
12. min,max	-> 返回最大/最小	->  .mapToDouble(Sku::getTotalPrice).max()
13. count -> 计数


```java
public class StreamOperation {
    List<Sku> skuList;

    @Before
    public void init() {
        skuList = CartService.getCartSkuList();
    }


    /**
     * 过滤
     */
    @Test
    public void filterTest() {
        skuList.stream()
                // filter 过滤
                .filter(sku -> SkuCategoryEnum.BOOKS.equals(sku.getSkuCategory()))
                .forEach(item -> System.out.println(JSON.toJSONString(item, true)));
    }


    /**
     * 有个容器，接收自己选取的内容
     */
    @Test
    public void mapTest() {
        skuList.stream()
                // map 容器
                .map(sku -> sku.getSkuName())
                .forEach(item -> System.out.println(JSON.toJSONString(item, true)));
    }


    /**
     * 接收一个新的流对象
     */
    @Test
    public void flatMapTest() {
        skuList.stream()
                // flatMap
                .flatMap(sku -> Arrays.stream(sku.getSkuName().split("")))
                .forEach(item -> System.out.println(JSON.toJSONString(item, true)));
    }

    /**
     * 也是循环，但是循环之后还可以进行其他循环操作
     */
    @Test
    public void peek() {
        skuList.stream()
                // peek
                .peek(sku -> System.out.println(sku.getSkuName()))
                .forEach(item -> System.out.println(JSON.toJSONString(item, true)))
               ;
    }


    /**
     * 流合并
     */
    @Test
    public void concat() {

        List<Integer> list1 = Lists.newArrayList(1, 2, 3);
        List<Integer> list2 = Lists.newArrayList(4, 5, 6);
        Stream<Integer> concat = Stream.concat(list1.stream(), list2.stream());

        concat.forEach(System.out::println);
    }

    /**
     * 排序
     */
    @Test
    public void sortTest() {
        skuList.stream()
                //sort
                .sorted(Comparator.comparing(Sku::getTotalPrice).reversed())
                .forEach(item -> System.out.println(JSON.toJSONString(item, true)));
    }

    /**
     * 去重
     */
    @Test
    public void distinctTest() {
        skuList.stream()
                // distinct
                .distinct()
                .forEach(item -> System.out.println(JSON.toJSONString(item, true)));
    }


    /**
     * 跳过  跳过下标之前的数据
     */
    @Test
    public void skipTest() {
        skuList.stream()
                //先排一下顺序
                .sorted(Comparator.comparing(Sku::getTotalPrice))
                // skip
                .skip(3)
                .forEach(item -> System.out.println(JSON.toJSONString(item, true)));
    }

    /**
     * 截取 指定数字N前的N条
     */
    @Test
    public void limitTest() {
        skuList.stream()
                .sorted(Comparator.comparing(Sku::getTotalPrice))
                // limit
                .limit(3)
                .forEach(item -> System.out.println(JSON.toJSONString(item, true)));
    }


    /**
     * 所有配陪
     */
    @Test
    public void allMatchTest() {
        boolean match = skuList.stream()
                .peek(sku -> System.out.println(sku.getSkuName()))
                // allMatch
                .allMatch(sku -> sku.getTotalPrice() > 100);
        System.out.println(match);
    }


    /**
     * 任意匹配
     */
    @Test
    public void anyMatchTest() {
        boolean match = skuList.stream()
                .peek(sku -> System.out.println(sku.getSkuName()))
                // anyMatch
                .anyMatch(sku -> sku.getTotalPrice() > 100);
        System.out.println(match);
    }

    /**
     * 没有一个匹配
     */
    @Test
    public void noneMatchTest() {
        boolean match = skuList.stream()
                .peek(sku -> System.out.println(sku.getSkuName()))
                // noneMatch
                .noneMatch(sku -> sku.getTotalPrice() > 10_000);
        System.out.println(match);
    }

    /**
     * 查找第一个
     */
    @Test
    public void findFirstTest() {
        Optional<Sku> optional = skuList.stream()
                .peek(sku -> System.out.println(sku.getSkuName()))
                // findFirst
                .findFirst();
        System.out.println(JSON.toJSONString(optional.get(), true));
    }


    /**
     * 查找任意
     */
    @Test
    public void findAnyTest() {
        Optional<Sku> optional = skuList.stream()
                .peek(sku -> System.out.println(sku.getSkuName()))
                // findAny
                .findAny();
        System.out.println(JSON.toJSONString(optional.get(), true));
    }


    /**
     *返回你定义的容器map  里面的参数中最大的值
     */
    @Test
    public void maxTest() {
        OptionalDouble optionalDouble = skuList.stream()
                // 获取总价
                .mapToDouble(Sku::getTotalPrice)
                .max();
        System.out.println(optionalDouble.getAsDouble());
    }
    @Test
    public void minTest() {
        OptionalDouble optionalDouble = skuList.stream()
                // 获取总价
                .mapToDouble(Sku::getTotalPrice)
                .min();
        System.out.println(optionalDouble.getAsDouble());
    }


    /**
     * 元素总数
     */
    @Test
    public void countTest() {
        long count = skuList.stream().count();
        System.out.println(count);
    }
}

```