### 01 依赖版本
```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.3.2.RELEASE</version>
	<relativePath/>
</parent>

<spring-cloud.version>Hoxton.SR10</spring-cloud.version>
<spring-cloud-alibaba.version>2.2.5.RELEASE</spring-cloud-alibaba.version>

<!--spring security-->
<dependency>
	<groupId>org.springframework.security.oauth.boot</groupId>
	<artifactId>spring-security-oauth2-autoconfigure</artifactId>
	<version>2.1.2.RELEASE</version>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-jwt</artifactId>
		</exclusion>
	</exclusions>
</dependency>

<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-jwt</artifactId>
	<version>1.1.1.RELEASE</version>
</dependency>
```