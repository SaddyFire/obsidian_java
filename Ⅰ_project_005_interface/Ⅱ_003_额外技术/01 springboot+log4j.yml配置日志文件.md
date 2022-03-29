##### 01 依赖
```xml
	<!-- 配置 log4j2 -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-log4j2</artifactId>
	</dependency>

	<!-- 加上这个才能辨认到log4j2.yml文件 -->
	<dependency>
		<groupId>com.fasterxml.jackson.dataformat</groupId>
		<artifactId>jackson-dataformat-yaml</artifactId>
	</dependency>

	<!-- 配置 log4j2 -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-log4j2</artifactId>
	</dependency>
	
	<!-- 加上这个才能辨认到log4j2.yml文件 -->
	<dependency>
		<groupId>com.fasterxml.jackson.dataformat</groupId>
		<artifactId>jackson-dataformat-yaml</artifactId>
	</dependency>
```

##### 02 log4j2.yml
`resources/log4j2.yml`
```yml
# 共有8个级别，按照从低到高为：ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < OFF。
Configuration:
  status: warn
  monitorInterval: 30
  Properties: # 定义全局变量
    Property: # 缺省配置（用于开发环境）。其他环境需要在VM参数中指定，如下：
      #测试：-Dlog.level.console=warn -Dlog.level.xjj=trace
      #生产：-Dlog.level.console=warn -Dlog.level.xjj=info
      - name: log.level.console
        value: info
      - name: log.path
        value: log
      - name: project.name
        value: opendoc
      - name: log.pattern
        value: "%d{yyyy-MM-dd HH:mm:ss.SSS} -%5p ${PID:-} [%15.15t] %-30.30C{1.} : %m%n"
  Appenders:
    Console:  #输出到控制台
      name: CONSOLE
      target: SYSTEM_OUT
      PatternLayout:
        pattern: ${log.pattern}
    #   启动日志
    RollingFile:
      - name: ROLLING_FILE
        fileName: ${log.path}/${project.name}.log
        filePattern: "${log.path}/historyRunLog/$${date:yyyy-MM}/${project.name}-%d{yyyy-MM-dd}-%i.log.gz"
        PatternLayout:
          pattern: ${log.pattern}
        Filters:
          #        一定要先去除不接受的日志级别，然后获取需要接受的日志级别
          ThresholdFilter:
            - level: error
              onMatch: DENY
              onMismatch: NEUTRAL
            - level: info
              onMatch: ACCEPT
              onMismatch: DENY
        Policies:
          TimeBasedTriggeringPolicy:  # 按天分类
            modulate: true
            interval: 1
        DefaultRolloverStrategy:     # 文件最多100个
          max: 100
      #   平台日志
      - name: PLATFORM_ROLLING_FILE
        ignoreExceptions: false
        fileName: ${log.path}/platform/${project.name}_platform.log
        filePattern: "${log.path}/platform/$${date:yyyy-MM}/${project.name}-%d{yyyy-MM-dd}-%i.log.gz"
        PatternLayout:
          pattern: ${log.pattern}
        Policies:
          TimeBasedTriggeringPolicy:  # 按天分类
            modulate: true
            interval: 1
        DefaultRolloverStrategy:     # 文件最多100个
          max: 100
      #   业务日志
      - name: BUSSINESS_ROLLING_FILE
        ignoreExceptions: false
        fileName: ${log.path}/bussiness/${project.name}_bussiness.log
        filePattern: "${log.path}/bussiness/$${date:yyyy-MM}/${project.name}-%d{yyyy-MM-dd}-%i.log.gz"
        PatternLayout:
          pattern: ${log.pattern}
        Policies:
          TimeBasedTriggeringPolicy:  # 按天分类
            modulate: true
            interval: 1
        DefaultRolloverStrategy:     # 文件最多100个
          max: 100
      #   错误日志
      - name: EXCEPTION_ROLLING_FILE
        ignoreExceptions: false
        fileName: ${log.path}/exception/${project.name}_exception.log
        filePattern: "${log.path}/exception/$${date:yyyy-MM}/${project.name}-%d{yyyy-MM-dd}-%i.log.gz"
        ThresholdFilter:
          level: error
          onMatch: ACCEPT
          onMismatch: DENY
        PatternLayout:
          pattern: ${log.pattern}
        Policies:
          TimeBasedTriggeringPolicy:  # 按天分类
            modulate: true
            interval: 1
        DefaultRolloverStrategy:     # 文件最多100个
          max: 100
      #   DB 日志
      - name: DB_ROLLING_FILE
        ignoreExceptions: false
        fileName: ${log.path}/db/${project.name}_db.log
        filePattern: "${log.path}/db/$${date:yyyy-MM}/${project.name}-%d{yyyy-MM-dd}-%i.log.gz"
        PatternLayout:
          pattern: ${log.pattern}
        Policies:
          TimeBasedTriggeringPolicy:  # 按天分类
            modulate: true
            interval: 1
        DefaultRolloverStrategy:     # 文件最多100个
          max: 100
  Loggers:
    Root:
      level: info
      AppenderRef:
        - ref: CONSOLE
        - ref: ROLLING_FILE
        - ref: EXCEPTION_ROLLING_FILE
    Logger:
      - name: platform
        level: info
        additivity: false
        AppenderRef:
          - ref: CONSOLE
          - ref: PLATFORM_ROLLING_FILE
      - name: bussiness
        level: info
        additivity: false
        AppenderRef:
          - ref: BUSSINESS_ROLLING_FILE
      - name: exception
        level: debug
        additivity: true
        AppenderRef:
          - ref: EXCEPTION_ROLLING_FILE
      - name: db
        level: info
        additivity: false
        AppenderRef:
          - ref: DB_ROLLING_FILE
#    监听具体包下面的日志
#    Logger: # 为com.xjj包配置特殊的Log级别，方便调试
#      - name: com.xjj
#        additivity: false
#        level: ${sys:log.level.xjj}
#        AppenderRef:
#          - ref: CONSOLE
#          - ref: ROLLING_FILE
```

##### 03 application.yml 添加配置
```yml
# log4j2
logging:
  config: classpath:log4j2.yml
```

##### 04 不同日志枚举类
`package com.consmation.demo.model.enums;`
```java
/**
 * 
 * 本地日志枚举
 * 由于配置了4个文件存放不同日志，分别为平台日志（${project.name}_platform.log）、 业务日志（${project.name}_bussiness.log）、错误日志（${project.name}_exception.log）、DB 日志（${project.name}_db.log），
 * 文件夹外面为运行日志，不同文件日志级别不一样，因此程序员在开发时候需要注意引入不同日志，
 * 也就是说开发出现的日志，程序员可以进行分类，分别调用公共方法即可。
 * 公共类编辑如下；
 * 
 * @author saddyfire
 *
 */
public enum LogEnum {
 
	BUSSINESS("bussiness"),PLATFORM("platform"),DB("db"),EXCEPTION("exception");
 
 
    private String category;
 
 
    LogEnum(String category) {
        this.category = category;
    }
 
    public String getCategory() {
        return category;
    }
 
    public void setCategory(String category) {
        this.category = category;
    }
    
}
```

##### 05 不同日志工具类util编辑
`package com.consmation.demo.utils;`
```java
import com.consmation.demo.model.enums.LogEnum;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @author SaddyFire
 * @date 2022/3/29
 * @TIME:20:24
 */
public class LogUtils {

    /**
     * 获取业务日志logger
     *
     * @return
     */
    public static Logger getBussinessLogger() {
        return LoggerFactory.getLogger(LogEnum.BUSSINESS.getCategory());
    }

    /**
     * 获取平台日志logger
     *
     * @return
     */
    public static Logger getPlatformLogger() {
        return LoggerFactory.getLogger(LogEnum.PLATFORM.getCategory());
    }

    /**
     * 获取数据库日志logger
     *
     * @return
     */
    public static Logger getDBLogger() {
        return LoggerFactory.getLogger(LogEnum.DB.getCategory());
    }


    /**
     * 获取异常日志logger
     *
     * @return
     */
    public static Logger getExceptionLogger() {
        return LoggerFactory.getLogger(LogEnum.EXCEPTION.getCategory());
    }

}
```
