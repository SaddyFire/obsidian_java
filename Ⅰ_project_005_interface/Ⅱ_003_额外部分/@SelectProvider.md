## 01 mapper层
**mapper文件中写入@SelectProvider注解方法**

```java
public interface AuditResultMapper extends Mapper<AuditResult> {
 
    @SelectProvider(type = SqlProvider.class, method = "getDataSql")
    List<AuditResultResponseVo> getResultData(AuditResultRequestVo vo);
}
```

## 02 provider层
**新建SqlProvider类**

```java
    /**
     * @description:  provider方法
     */    
public String getDataSql(AuditResultRequestVo vo) {
 
        StringBuffer stringBuffer = new StringBuffer();
        stringBuffer.append("SELECT  DATE_FORMAT(a.begin_time,'%Y-%m-%d') AS beginTime,  count(1) AS dayCount " +
                "FROM  audit_result a WHERE 1=1 ");
        if (!StringUtils.isEmpty(vo.getEndTime())) {
            stringBuffer.append(" and a.begin_time <= '" + vo.getEndTime() +"'");
        }
 
        if (!StringUtils.isEmpty(vo.getStartTime())) {
            stringBuffer.append(" and a.begin_time >= '" + vo.getStartTime() +"'" );
        }
 
        stringBuffer.append(" GROUP BY beginTime ORDER BY  beginTime DESC ");
 
        return stringBuffer.toString();
 
    }
```

## 03 controller层

**注入mapper接口，调用数据查询方法**
```java
List<AuditResultResponseVo> listResult = auditResultMapper.getResultData(vo);
```
