# REST
REST（Representational State Transfer），表述性状态转换，它是一种软件架构风格

传统URL：
- 定义比较复杂，而且将资源的访问行为对外暴露出来了。
```html
http://localhost:8080/user/getById?id=1     GET：查询id为1的用户
http://localhost:8080/user/saveUser         POST：新增用户
http://localhost:8080/user/updateUser       POST：修改用户
http://localhost:8080/user/deleteUser?id=1  GET：删除id为1的用户
```

基于REST风格URL:
- 通过URL定位要操作的资源，通过HTTP动词(请求方式)来描述具体的操作。
```html
http://localhost:8080/users/1  GET：查询id为1的用户
http://localhost:8080/users    POST：新增用户
http://localhost:8080/users    PUT：修改用户
http://localhost:8080/users/1  DELETE：删除id为1的用户
```

在REST风格的URL中，通过四种请求方式，来操作数据的增删改查:
- GET 查询
- POST 新增
- PUT 修改
- DELETE 删除

# 开发规范
## 项目结构

## 开发流程

## 1. 写持久层 - Mapper
- `@Mapper`
- 接入数据库
- 通过 Mybatis 注解对数据库进行操作

## 2. 写服务层 - Service
- `@Service`
- 先写抽象接口
- 再写具体类
- 注入需要的 Mapper 容器
- 调用 Mapper 容器中对应的方法

## 3. 写接口 - Controller
- `@RestController`
- 注入需要的 Service 容器
- 在方法上方标注请求路径
- 在方法中调用 Service 容器中对应的方法
- 返回 Result
#### 请求路径
- 在Spring当中为了简化请求路径的定义，可以把公共的请求路径，直接抽取到类上，在类上加一个注解@RequestMapping，并指定请求路径，如"/depts"。
- 一个完整的请求路径，应该是类上@RequestMapping的value属性 + 方法上方的 @RequestMapping的value属性
![[request-mapping.png]]

### 请求方法
- `@GetMapping` 查询
- `@DeleteMapping` 删除
- `@PutMapping` 修改
	- 因为前端传入的是JSON，方法参数需要 `@RequestBody`
- `@PostMapping` 新增
	- 因为前端传入的是JSON，方法参数需要 `@RequestBody`

####  统一响应结果 Result
前后端工程在进行交互时，使用统一响应结果 Result
```Java
package com.itheima.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Result {
    private Integer code;//响应码，1 代表成功; 0 代表失败
    private String msg;  //响应信息 描述字符串
    private Object data; //返回的数据

    //增删改 成功响应
    public static Result success(){
        return new Result(1,"success",null);
    }
    //查询 成功响应
    public static Result success(Object data){
        return new Result(1,"success",data);
    }
    //失败响应
    public static Result error(String msg){
        return new Result(0,msg,null);
    }
}
```

# 上传文件
- Spring 中提供了一个 API `MultipartFile`，可以使用这个 API 来接收到上传的文件

```Java
@Slf4j  
@RestController  
public class UploadController {  
  
    @PostMapping("/upload")  
    public Result upload(String username, Integer age, MultipartFile image) {  
        log.info("文件上传: {},{},{}", username,age,image);  
        return Result.success();  
    }  
}
```

>[!info] 表单提交的数据分别存储在不同的临时文件中
>当我们程序运行完毕之后，这个临时文件会自动删除。所以，我们如果想要实现文件上传，需要将这个临时文件，要转存到我们的磁盘目录中。

## 本地存储
- 在服务器本地磁盘上创建目录，如`/images`
- 使用MultipartFile类提供的API方法，把临时文件转存到本地磁盘目录下
- MultipartFile 常见方法：
	- `String getOriginalFilename()`; //获取原始文件名
	- `void transferTo(File dest);` //将接收的文件转存到磁盘文件中
	- `long getSize();`//获取文件的大小，单位：字节
	- `byte[] getBytes();`//获取文件内容的字节数组
	- `InputStream getInputStream();` //获取接收到的文件内容的输入流
## 文件命名
通过postman测试，我们发现文件上传是没有问题的。但是由于我们是使用原始文件名作为所上传文件的存储名字，当我们再次上传一个名为1.jpg文件时，发现会把之前已经上传成功的文件覆盖掉。

解决方案：保证每次上传文件时文件名都唯一的（使用UUID获取随机文件名）

# YML配置文件
## 基本语法
- 大小写敏感
- 数值前边必须有空格，作为分隔符
- 使用缩进表示层级关系，缩进时，不允许使用Tab键，只能用空格（idea中会自动将Tab转换为空格）
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
- `#`表示注释，从这个字符一直到行尾，都会被解析器忽略

两种常见类型
- 定义对象或 Map 集合
```YAML
user:
  name: zhangsan
  age: 18
  password: 123456
```

- 定义数组、list 或 set 集合
```YAML
hobby: 
  - java
  - game
  - sport
```
## @ConfigurationProperties
使用 `@Value("${key}")`这种方式来注解存在的问题:
- @Value注解只能一个一个的进行外部属性的注入
- 如果说需要注入的属性较多(例：需要20多个参数数据)，我们写起来就会比较繁琐。

Spring提供的简化方式套路 `@ConfigurationProperties`：
- 可以批量的将外部的属性配置注入到bean对象的属性中

实现步骤：
1. 需要创建一个实现类，且实体类中的属性名和配置文件当中key的名字必须要一致。比如：配置文件当中叫endpoints，实体类当中的属性也得叫endpoints，另外实体类当中的属性还需要提供 getter / setter方法
2. 需要将实体类交给Spring的IOC容器管理，成为IOC容器当中的bean对象
3. 在实体类上添加`@ConfigurationProperties`注解，并通过 `prefix` 属性来指定配置参数项的前缀

在 `utils/` 中创建实体类
```Java
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/*阿里云OSS相关配置*/
@Data
@Component
@ConfigurationProperties(prefix = "aliyun.oss")
public class AliOSSProperties {
    //区域
    private String endpoint;
    //身份ID
    private String accessKeyId ;
    //身份密钥
    private String accessKeySecret ;
    //存储空间
    private String bucketName;
}
```

引入依赖
```Java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
```

在需要使用这些外部属性的类别里注入容器
```Java
public class AliOSSUtils {  
   @Autowired  
   private AliOSSProperties aliOSSProperties;
   
   // 省略方法
}
```