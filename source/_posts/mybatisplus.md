---
title: mybatisplus
date: 2023-10-16 09:46:48
tags:
---

## Mybatis-plus

## 1. 什么是MyBatis-Plus

MyBatis-Plus 是一个 Mybatis 增强版工具，在 MyBatis 上扩充了其他功能没有改变其基本功能，为了简化开发提交效率而存在。

官网文档地址：　https://mp.baomidou.com/guide/



## 2. springboot中快速使用Mybatis-plus

### 使用插件

使用 IDEA 安装一个 mybatis-plus 插件

![01.插件](/01.插件.png)

### 增加依赖

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>6.0.6</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.24</version>
</dependency>
```



### 配置application.yml

```yaml
server:
  port: 8080
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/java160?serverTimezone=Asia/Shanghai&useSSL=false
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource

    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
mybatis:
  type-aliases-package: com.hyy.entity
```

### 增加Config类

```java
@Configuration
public class MyConfig {
    @Bean("dataSource")
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druid() {
        return new DruidDataSource();
    }
}
```

### 增加实体类

```java
@Data
public class Teacher {
    String tno;
    String tname;
    String tsex;
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    Date tbirthday;
    String prof;
    String depart;
}
```

### 编写Mapper类

```java
public interface TeacherMapper extends BaseMapper<Teacher> {
}
```

### 修改启动类

增加@MapperScan注解

```java
@SpringBootApplication
@MapperScan(basePackages = "com.hyy.dao")
public class Base04MybatisplusApplication {

    public static void main(String[] args) {
        SpringApplication.run(Base04MybatisplusApplication.class, args);
    }

}
```

### 编写测试控制器

此控制器跳开service，直接调用了Mapper

```java
@RestController
public class TeacherController {
    @Autowired
    TeacherMapper teacherMapper;
    @RequestMapping("/getall")
    public List<Teacher> selectAll(){
        return teacherMapper.selectList(null);
    }
}
```



### 配置日志

```
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```



## 常用注解

@TableName：用于表名，放在entity上面，value值是数据库表名（主要用于pojo和数据库表名不一致的情况下）

@TableId：用于定义表的主键，其value是表的主键column名，type是指主键类型

@TableField：用于定义表的非主键字段。

​	value：指定数据库表中的cloumn名

​	exist：用于表明当前的field是否是数据库表的一个属性，会影响sql语句

​	fill：用于指定数据的自动填充策略，一般用于修改时间或者创建时间，默认不填充

@TableLogic：用于定义表的逻辑删除字段。

​	value：用于定义未删除状态的值

​	delval：用于定义删除状态的值

​	

## 代码生成器

## 新版（3.5.1之后）

### 依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.3</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.5.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.velocity</groupId>
            <artifactId>velocity-engine-core</artifactId>
            <version>2.3</version>
        </dependency>
```



### 配置

```java
@Test
    void mybatisGenerator() {
        String url = "jdbc:mysql://localhost:3306/authc?serverTimezone=UTC&useSSL=false";
        //总体配置
         FastAutoGenerator fastAutoGenerator = FastAutoGenerator.create(url, "root", "root")
                .globalConfig(builder -> {
                    builder.author("mamp") // 设置作者
                            //.commentDate(LocalDateTime.now().toString())
                            //.enableSwagger() // 开启 swagger 模式
                            .fileOverride()
                            .outputDir("D://"); // 指定输出目录
                });
        //包配置
        fastAutoGenerator.packageConfig(builder -> {
                    builder.parent("net")
                            .moduleName("system")
                            .controller("controller")
                            .service("service")
                            .entity("domain")
                            .mapper("mapper")
                            .serviceImpl("impl");
                });
        //实体策略
        fastAutoGenerator.strategyConfig(builder -> {
            builder.entityBuilder()
                    .naming(NamingStrategy.underline_to_camel)
                    .columnNaming(NamingStrategy.underline_to_camel);
            builder.controllerBuilder().enableRestStyle();
        });
        //执行
        fastAutoGenerator.execute();
    }
```



### 模板

#### 定义模板

定义模板并放在resources/templates目录下，以下为controller.java.vm

```velocity
package ${package.Controller};

import org.springframework.web.bind.annotation.RequestMapping;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import ${package.Entity}.${entity};
import ${package.Service}.${table.serviceName};
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.util.List;

#if(${restControllerStyle})
import org.springframework.web.bind.annotation.RestController;
#else
import org.springframework.stereotype.Controller;
#end
#if(${superControllerClassPackage})
import ${superControllerClassPackage};
#end

/**
 * <p>
 * $!{table.comment} 前端控制器
 * </p>
 *
 * @author ${author}
 * @since ${date}
 */
#if(${restControllerStyle})
@RestController
#else
@Controller
#end
@RequestMapping("#if(${package.ModuleName})/${package.ModuleName}#end/#if(${controllerMappingHyphenStyle})${controllerMappingHyphen}#else${table.entityPath}#end")
#if(${kotlin})
class ${table.controllerName}#if(${superControllerClass}) : ${superControllerClass}()#end

#else
#if(${superControllerClass})
public class ${table.controllerName} extends ${superControllerClass} {
#else
public class ${table.controllerName} {

        }
#end

    @Resource
    ${table.serviceName} #serviceName();

    @GetMapping("list")
    public List<${entity}> doList(){
            QueryWrapper<${entity}> query = new QueryWrapper<>();
            query.likeLeft("phone","23")
            .like("email","2189");
            List<${entity}> list = #serviceName().list(query);
            return list;
    }

    @GetMapping("findOne/{id}")
    public ${entity} findOne(@PathVariable("id") Long id){
        return #serviceName().getById(id);
    }

    @DeleteMapping("delete/{id}")
    public void deleteOne(@PathVariable("id") Long id){
        ${table.serviceName.substring(0,1).toLowerCase()}${table.serviceName.substring(1)}.removeById(id);
    }


}
#end

#macro(serviceName)
    ${table.serviceName.substring(0,1).toLowerCase()}${table.serviceName.substring(1)}
#end

```

#### 修改生成代码

```java
@Test
    void mybatisGenerator() {
         ...
        //resourceLoader使用的是ClasspathResourceLoader，所以只需要写classpath下的目录即可，不需要classpath前缀
        String controllerTemplate="templates/controller.java";
        fastAutoGenerator.templateConfig(builder -> {
            builder.controller(controllerTemplate);
        });
        fastAutoGenerator.execute();
    }
```



## 旧版（3.4.x)

使用代码生成器能够方便的生成所需要的代码

注意在主类中需要增加@EnableAutoConfiguration或者给@SpringBootApplication注解添加属性exclude = {DataSourceAutoConfiguration.class}

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class GeneratorApplication {
	public static void main(String[] args) {
        SpringApplication.run(GeneratorApplication.class, args);
    }
}
```



### 增加依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>
<!-- 默认的引擎模板-->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.3</version>
</dependency>
```



### 创建代码生成器

```java
AutoGenerator mpg = new AutoGenerator();
```



### 全局配置

```java
// Step2：全局配置
GlobalConfig gc = new GlobalConfig();
// 填写代码生成的目录(需要修改)
String projectPath = "D:\\springbootProject\\springbootsample\\base05-mybatisplus-generator";
// 拼接出代码最终输出的目录
gc.setOutputDir(projectPath + "/src/main/java");
// 配置开发者信息（可选）（需要修改）
gc.setAuthor("mamp");
// 配置是否打开目录，false 为不打开（可选）
gc.setOpen(false);
// 实体属性 Swagger2 注解，添加 Swagger 依赖，开启 Swagger2 模式（可选）
//gc.setSwagger2(true);
// 重新生成文件时是否覆盖，false 表示不覆盖（可选）
gc.setFileOverride(false);
// 配置主键生成策略，此处为 ASSIGN_ID（可选）
gc.setIdType(IdType.ASSIGN_ID);
// 配置日期类型，此处为 ONLY_DATE（可选）
gc.setDateType(DateType.ONLY_DATE);
// 默认生成的 service 会有 I 前缀
gc.setServiceName("%sService");
mpg.setGlobalConfig(gc);
```

### 设置数据源

```java
// Step3：数据源配置（需要修改）
DataSourceConfig dsc = new DataSourceConfig();
// 配置数据库 url 地址
String url="jdbc:mysql://localhost:3306/java160?"
        +"serverTimezone=Asia/Shanghai&useSSL=false";
System.out.println(url);
dsc.setUrl(url);
// 配置数据库驱动
dsc.setDriverName("com.mysql.cj.jdbc.Driver");
// 配置数据库连接用户名
dsc.setUsername("root");
// 配置数据库连接密码
dsc.setPassword("root");
mpg.setDataSource(dsc);
```

### 设置包

```java
// Step:4：包配置
PackageConfig pc = new PackageConfig();
// 配置父包名（需要修改）
pc.setParent("com.hyy");
// 配置模块名（需要修改）
pc.setModuleName("system");
// 配置 entity 包名
pc.setEntity("entity");
// 配置 mapper 包名
pc.setMapper("mapper");
// 配置 service 包名
pc.setService("service");
// 配置 controller 包名
pc.setController("controller");
mpg.setPackageInfo(pc);
```

### 设置生成策略

```java
// Step5：策略配置（数据库表配置）
StrategyConfig strategy = new StrategyConfig();
// 指定表名（可以同时操作多个表，使用 , 隔开）（需要修改）
strategy.setInclude("teacher");
// 配置数据表与实体类名之间映射的策略
strategy.setNaming(NamingStrategy.underline_to_camel);
// 配置数据表的字段与实体类的属性名之间映射的策略
strategy.setColumnNaming(NamingStrategy.underline_to_camel);
// 配置 lombok 模式
strategy.setEntityLombokModel(true);
// 配置 rest 风格的控制器（@RestController）
strategy.setRestControllerStyle(true);
// 配置驼峰转连字符
strategy.setControllerMappingHyphenStyle(true);
// 配置表前缀，生成实体时去除表前缀
// 此处的表名为 test_mybatis_plus_user，模块名为 test_mybatis_plus，去除前缀后剩下为 user。
//        strategy.setTablePrefix(pc.getModuleName() + "_");
mpg.setStrategy(strategy);
```



### 执行代码生成操作

```
mpg.execute();
```





## 其他功能

## 自动填充数据

给POJO的属性@TableField上增加fill属性

定义一个实现MetaObjectHandler 接口的组件（需要能够扫描到）

- POJO属性

  ```java
  @Data
  @EqualsAndHashCode(callSuper = false)
  public class Teacher implements Serializable {
  
      private static final long serialVersionUID = 1L;
  
      //.....其他属性
  
      @TableField(value = "createTime",fill = FieldFill.INSERT)
      private Date createtime;
  
      @TableField(value = "updateTime",fill = FieldFill.INSERT_UPDATE)
      private Date updatetime;
  
  }
  
  ```

  

- MetaObjectHandler 的实现类，定义自动填充规则

  ```java
  @Component
  public class BaseObjectHandler implements MetaObjectHandler {
      @Override
      public void insertFill(MetaObject metaObject) {
          this.strictInsertFill(metaObject, "createtime", Date.class, new Date());
          this.strictInsertFill(metaObject, "updatetime", Date.class, new Date());
      }
  
      @Override
      public void updateFill(MetaObject metaObject) {
          this.strictInsertFill(metaObject, "updatetime", Date.class, new Date());
      }
  }
  ```

  

## 逻辑删除

支持的数据类型为Integer，Boolean以及Date类型。

操作方法：给实体类的对象增加@TableLogic(value="0",delval = "1")注解

- value：正常有效数据值
- delval：删除时的值

使用 mybatis-plus 封装好的方法时，会自动添加逻辑删除的功能。若是自定义的 sql 语句，需要手动添加逻辑。

### 全局设置（3.5.1以后版本）

在application.yml文件中增加以下配置，并在entity的指定属性上增加@TableLogic注解

```java
mybatis-plus  
  global-config:
    db-config:
      logic-delete-field: del_flag # 全局逻辑删除的实体字段名
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)

```

entity

```java
@Data
@TableName(value="sample")
public class Sample implements Serializable {
	...
    @TableLogic
    private Integer delFlag;
}

```



### 单个设置

使用@TableLogic的value和delValue来设置

```java
@Data
@EqualsAndHashCode(callSuper = false)
public class Teacher implements Serializable {

    private static final long serialVersionUID = 1L;

     。。。

         
    @TableLogic(value="0",delval = "1")  //增加此字段和注解
    @TableField("deleteFlag")
    private Integer deleteflag;

}
```



## 分页插件

### 注入mybatisplus分页插件

```
@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

### 使用mybatis提供的方法

mybatis提供了一系列分页的方法，例如page

```java
@GetMapping("page")
public Page<Sample> findPage(){
    Page<Sample> page = new Page<>();
    page.setCurrent(1);
    page.setSize(2);
    Page<Sample> page1 = sampleService.page(page);
    return page1;
}
```



### 使用自定义方法

- 修改mapper

  ```java
  @Mapper
  public interface SampleMapper extends BaseMapper<Sample> {
  
      IPage<Sample> selectPageVo(IPage<?> page,  Sample sample);
  
  }
  
  
  //mapper.xml
  <select id="selectPageVo" resultType="net.wanho.base01.entity.Sample">
          select * from sample
          where name like concat('%',#{sample.name},'%')
      </select>
  ```

  

- 修改service

  ```java
  public interface ISampleService extends IService<Sample> {
      IPage<Sample> selectPageVo(IPage<?> page, Sample sample);
  }
  
  
  @Service
  public class SampleServiceImpl extends ServiceImpl<SampleMapper, Sample> implements ISampleService {
  
      @Override
      public IPage<Sample> selectPageVo(IPage<?> page, Sample sample) {
          SampleMapper baseMapper = this.getBaseMapper();
          return baseMapper.selectPageVo(page,sample);
      }
  }
  ```

  

- 测试处理器

  ```
  @GetMapping("mypage")
      public Page<Sample> findMyPage(){
          Page<Sample> page = new Page<>();
          page.setCurrent(1);
          page.setSize(2);
          Sample sample = new Sample();
          sample.setName("test");
          IPage<Sample> page1 = sampleService.selectPageVo(page,sample);
  
        return (Page<Sample>)page1;
      }
  ```
  
  



## 乐观锁（3.5.1）

## 条件构造器

QueryWrapper：查询条件构造器

```java
QueryWrapper<Teacher> queryWrapper = new QueryWrapper<>();
queryWrapper
            .select("tno", "tname", "tbirthday")
            .eq("tno", '702')
            .like("tname", "j");
 teacherService
            .list(queryWrapper)
            .forEach(System.out::println);
```