## 简介

动态数据源插件,可快速集成到基于springboot的工程项目中。

## 快速集成
1. 
```bash
git clone https://github.com/lufengc/dynamic-datasource.git
cd dynamic-datasource/
mvn clean install -Dmaven.test.skip
```
2. 引入依赖
```xml
<dependency>
    <groupId>com.dsjk</groupId>
    <artifactId>dynamic-datasource</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```
3. 配置数据源。application.yml
```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/backend-boot?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root
    # set pool
    initialSize: 1
    minIdle: 1
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 'x'
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxPoolPreparedStatementPerConnectionSize: 20
    filters: stat
    monitor:
      open: true
      allow: 127.0.0.1
      deny:
      loginUsername: admin
      loginPassword: admin
      
custom.datasource:
  names: ds1,ds2
  ds1.type: com.alibaba.druid.pool.DruidDataSource
  ds1.driverClassName: com.mysql.jdbc.Driver
  ds1.url: jdbc:mysql://localhost:3306/ds1?useUnicode=true&characterEncoding=UTF-8&useSSL=false
  ds1.username: root
  ds1.password: root

  ds2.type: com.alibaba.druid.pool.DruidDataSource
  ds2.driverClassName: com.mysql.jdbc.Driver
  ds2.url: jdbc:mysql://localhost:3306/ds2?useUnicode=true&characterEncoding=UTF-8&useSSL=false
  ds2.username: root
  ds2.password: root
```
> *如果服务类项目，如：dubbo服务，则排除web模块，去掉DruidAutoConfiguration中的拦截器即可*
4. 启动注册。在启动类上添加注解@Import({DynamicDataSourceRegister.class})，如：
```java
@Import({DynamicDataSourceRegister.class})
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
> *若启动类的扫描包路径无法扫描到，则需要在启动类上添加扫描路径，如：@ComponentScan(basePackages = "com.bdfint.datasource")*

5. 使用。指定数据源则在方法上使用注解@TargetDataSource("ds1"), 不指定则使用默认数据源，如：
```java
@Service
public class TAreaServiceImpl implements TAreaService {

    @Autowired
    private TAreaMapper tAreaMapper;
    @Autowired
    private AreaMapper AreaMapper;

    @Override
    @Transactional
    public void updateData() throws Exception {
        Area tArea = new Area();
        tArea.setId("1");
        tArea.setName("中国0");
        AreaMapper.updateByPrimaryKeySelective(tArea);
    }

    @Override
    @Transactional
    @TargetDataSource("ds1")
    public void updateDatads() throws Exception {
        TArea tArea = new TArea();
        tArea.setId("1");
        tArea.setName("中国ds1");
        tAreaMapper.updateByPrimaryKeySelective(tArea);
    }

}
```

## 版本说明
* springboot 1.5.6.RELEASE
* druid 1.1.3

## 开发环境
* Mac OS
* IntelliJ IDEA 2017.2.5
* jdk 1.8
* Maven 3.3.9
* Tomcat 8.5
* Mysql 5.7