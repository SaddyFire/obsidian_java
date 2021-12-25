**1. anyMatch的使用**
anyMatch(Predicate predicate) 根据条件进行匹配返回 boolean

```java

/**
 * 类描述：案例一  查找缺考的学生
 * 重点讲解：anyMatch的使用方式
 */
public class AnyMatchCase {

    /**
     * 考试成绩模型
     */
    @Data
    @AllArgsConstructor
    class ExamStudentScore {
        private String studentName;//学生姓名
        private Integer scoreValue;//成绩
        private String subject;//科目
    }

    Map<String, List<ExamStudentScore>> studentMap;//学生考试成绩

    @Before
    public void init() {
        studentMap = new HashMap<>();

        List<ExamStudentScore> zsScoreList = new ArrayList<>();
        zsScoreList.add(new ExamStudentScore("张三", 30, "CHINESE"));
        zsScoreList.add(new ExamStudentScore("张三", 40, "ENGLISH"));
        zsScoreList.add(new ExamStudentScore("张三", 50, "MATHS"));
        studentMap.put("张三", zsScoreList);

        List<ExamStudentScore> lsScoreList = new ArrayList<>();
        lsScoreList.add(new ExamStudentScore("李四", 80, "CHINESE"));
        lsScoreList.add(new ExamStudentScore("李四", null, "ENGLISH"));
        lsScoreList.add(new ExamStudentScore("李四", 100, "MATHS"));
        studentMap.put("李四", lsScoreList);

        List<ExamStudentScore> wwScoreList = new ArrayList<>();
        wwScoreList.add(new ExamStudentScore("王五", null, "CHINESE"));
        wwScoreList.add(new ExamStudentScore("王五", null, "ENGLISH"));
        wwScoreList.add(new ExamStudentScore("王五", 70, "MATHS"));
        studentMap.put("王五", wwScoreList);
    }

    @Test
    public void testFindStudent(){
        studentMap.forEach(
                (stuName,stuScores)->{
                    //方法一: 匹配分数为0
                    boolean b = stuScores.stream().anyMatch(s -> s.getScoreValue() == null);
                    if(b){
                        System.out.println(stuName + ":缺考");
                    }
                    //方法二: 传统forEach遍历
                    stuScores.forEach(s -> {
                        if(s.getScoreValue() == null){
                            System.out.println(stuName + "缺考 " + s.getSubject());
                        }
                    });
                });
    }

}

```