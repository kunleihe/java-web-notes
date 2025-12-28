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

```
springboot-web-demo
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com.example/
│   │   │       ├── controller/     # 控制层：处理 HTTP 请求 
│   │   │       │   ├── DeptController.java
│   │   │       │   ├── EmpController.java
│   │   │       │   └── UploadController.java
│   │   │       ├── mapper/         # 数据持久层：MyBatis Mapper 接口
│   │   │       │   ├── DeptMapper.java
│   │   │       │   └── EmpMapper.java
│   │   │       ├── pojo/           # 实体类层：普通 Java 对象
│   │   │       │   ├── Dept.java
│   │   │       │   ├── Emp.java
│   │   │       │   ├── PageBean.java
│   │   │       │   └── Result.java
│   │   │       ├── service/        # 业务逻辑层
│   │   │       │   ├── impl/       # 业务逻辑实现类
│   │   │       │   │   ├── DeptServiceImpl.java
│   │   │       │   │   └── EmpServiceImpl.java
│   │   │       │   ├── DeptService.java (Interface)
│   │   │       │   └── EmpService.java (Interface)
│   │   │       ├── utils/          # 工具类
│   │   │       │   ├── AliOSSProperties.java
│   │   │       │   └── AliOSSUtils.java
│   │   │       └── SpringbootWebDemoApplication.java # 项目启动类
│   │   └── resources/              # 资源文件
│   │       ├── com.example.mapper/ # MyBatis SQL 映射 XML 文件
│   │       ├── static/             # 静态资源（JS, CSS, 图片等）
│   │       ├── templates/          # 模板文件（如 Thymeleaf）
│   │       └── application.yml     # 项目核心配置文件
│   └── test/                       # 测试代码
└── pom.xml                         # Maven 项目依赖管理文件
```

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
```YML
spring:  
  datasource:  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    url: jdbc:mysql://localhost:3306/springboot-web-demo-db  
    username: root  
    password: root  
  servlet:  
    multipart:  
      max-file-size: 10MB  
      max-request-size: 100MB  
  
mybatis:  
  configuration:  
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  
    map-underscore-to-camel-case: true  
  
aliyun:  
  oss:  
    endpoint: https://oss-cn-hangzhou.aliyuncs.com  
    accessKeyId: yourAccessKey  
    accessKeySecret: yourAccessKeySecret  
    bucketName: yourBucketName
```
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
# 登录认证
登录校验
- 在服务器端接收到浏览器发送过来的请求之后，首先我们要对请求进行校验。
	- 先要校验一下用户登录了没有，如果用户已经登录了，就直接执行对应的业务操作就可以了
	- 如果用户没有登录，此时就不允许他执行相关的业务操作，直接给前端响应一个错误的结果，最终跳转到登录页面，要求他登录成功之后，再来访问对应的数据。

如何实现登录校验？
- 在用户登录成功后，需要将用户登录成功的信息存起来，记录用户已经登录成功的标记
- 在浏览器发起请求时，需要在服务端进行统一拦截，拦截后进行登录校验
- 我们程序中所开发的查询功能、删除功能、添加功能、修改功能，都需要使用以上套路进行登录校验。此时就会出现：相同代码逻辑，每个功能都需要编写，就会造成代码非常繁琐。
- 为了简化这块操作，我们可以使用一种技术：统一拦截技术。
- 通过统一拦截的技术，我们可以来拦截浏览器发送过来的所有的请求，拦截到这个请求之后，就可以通过请求来获取之前所存入的登录标记，在获取到登录标记且标记为登录成功，就说明员工已经登录了。如果已经登录，我们就直接放行(意思就是可以访问正常的业务接口了)。

## 会话技术
- 在web开发当中，会话指的就是浏览器与服务器之间的一次连接，我们就称为一次会话。
- 在用户打开浏览器第一次访问服务器的时候，这个会话就建立了，直到有任何一方断开连接，此时会话就结束了。
- 在一次会话当中，是可以包含多次请求和响应的。

> [!info] 会话是和浏览器关联的
> 当有三个浏览器客户端和服务器建立了连接时，就会有三个会话。当我们关闭浏览器之后，属于这个浏览器的这次会话就结束了。而如果我们是直接把web服务器关了，那么所有的会话就都结束了。

### 会话跟踪
- 一种维护浏览器状态的方法，服务器需要识别多次请求是否来自于同一浏览器，以便在同一次会话的多次请求间共享数据
- 三种会话跟踪技术
	- Cookie - 客户端会话跟踪技术
		- 数据储存在客户端浏览器当中
	- Session - 服务器端会话跟踪技术
		- 数据储存在服务端
	- 令牌技术
#### Cookie
- Cookie 是客户端会话跟踪技术，它是存储在客户端浏览器的，我们使用 cookie 来跟踪会话，我们就可以在浏览器第一次发起请求来请求服务器的时候，我们在服务器端来设置一个cookie。
	
Cookie的四个核心步骤
1. 服务器设置 Cookie
	- 当你第一次访问服务器（比如登录）时，服务器验证了你的身份，设置了Cookie
2. 服务器自动响应回浏览器（传输Cookie）
	- 服务器在给你返回“登录成功”的消息时，会在 HTTP 响应头（Response Headers）里自动带上这个 Cookie。
3. 浏览器自动存储Cookie
	- 浏览器看到响应头里有 `Set-Cookie`，就会自动把这个键值对存到电脑的硬盘或内存里。这个过程是**自动**的，不需要前端写代码去存。
4. 后续请求自动携带Cookie
	- 当你下一次访问该网站的其他页面（比如查看个人订单）时，浏览器会自动检查：_“我这里有没有这个网站的 Cookie？”_ 如果有，就自动塞进请求头（Request Headers）里发给服务器

优缺点
- 优点
	- HTTP协议中支持的技术（像Set-Cookie 响应头的解析以及 Cookie 请求头数据的携带，都是浏览器自动进行的，是无需我们手动操作的）
- 缺点
	- 移动端APP(Android、IOS)中无法使用Cookie
	- 不安全，用户可以自己禁用Cookie
	- Cookie不能跨域，而现在的项目基本都是前后端分离的
#### Session
- Session是服务器端的会话跟踪技术，是基于Cookie来实现的

Session的四个核心步骤
1. 第一次握手：创建 Session
	- 当你第一次请求服务器（比如登录）时：
		- 服务器会为这个空间生成一个唯一的编号，通常叫 **JSESSIONID**
		- 服务器通过 `Set-Cookie` 把这个编号发给你的浏览器。
2. 存储阶段：浏览器存编号
	- 浏览器收到响应后，会自动把 `JSESSIONID=xxxx` 存入 Cookie 中。
3. 后续请求：携带SessionID
	- 下次你访问服务器时，浏览器会自动在请求头的 Cookie 字段里带上这个编号：`Cookie: JSESSIONID=xxxx`。
4. 服务器识别
	- 服务器拿到编号后，去自己的内存里搜索并读取你的信息

优缺点
- 优点
	- Session是存储在服务端的，安全
- 缺点
	- 服务器集群环境下无法直接使用Session
	- 移动端APP(Android、IOS)中无法使用Cookie
	- 不安全，用户可以自己禁用Cookie
	- Cookie不能跨域，而现在的项目基本都是前后端分离的
#### 令牌技术 （最常用）
1. 登录验证（颁发令牌）
	- 用户登录时，服务器校验用户名密码正确。
	- 服务器把用户的 ID、角色等非敏感信息写在一张“纸”上，然后用只有服务器知道的密钥（Secret Key）在上面签名。
	- 并把这个生成的长字符串（Token）返回给浏览器。
2. 客户端存储
	- 浏览器拿到 Token 后，通常由前端开发人员通过代码存入 `localStorage` 或 `sessionStorage`（注意：这不像 Cookie 那样是自动的，需要代码操作）
3. 后续请求（携带令牌）
	- 以后每一次请求，前端都需要手动在 HTTP 的 **Authorization** 请求头里带上这个 Token。
	- 格式：`Authorization: Bearer <Token内容>`
4. 服务器校验
	- 服务端统一拦截请求之后，先来判断一下这次请求有没有把令牌带过来，如果没有带过来，直接拒绝访问，如果带过来了，还要校验一下令牌是否是有效。如果有效，就直接放行进行请求的处理。

优缺点
- 优点：
    - 支持PC端、移动端
    - 解决集群环境下的认证问题
    - 减轻服务器的存储压力（无需在服务器端存储）
- 缺点：需要自己实现（包括令牌的生成、令牌的传递、令牌的校验）
## JWT令牌
JWT全称：JSON Web Token （官网：[https://jwt.io/](https://jwt.io/)）

JWT的组成
- Header(头）， 记录令牌类型、签名算法等。 例如：{"alg":"HS256","type":"JWT"}
- Payload(有效载荷），携带一些自定义信息、默认信息等。 例如：{"id":"1","username":"Tom"}
- Signature(签名），防止Token被篡改、确保安全性。将header、payload，并加入指定秘钥，通过指定签名算法计算而来。

> [!info] 签名的目的
> 签名的目的就是为了防jwt令牌被篡改。在解析和校验JWT时，我们会取出Header 和 Payload，配合签名重新跑一遍加密算法。如果新算的签名和JWT自带的第三段签名完全一致，说明这个 Token 是我们签发的，没有被篡改过。

JWT是如何将原始的JSON格式数据，转变为字符串的呢？
- 通过对JSON数据进行 base64 编码

### 生成和校验
引入JWT依赖
```XML
<!-- JWT依赖-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

录下发令牌
1. 生成令牌：在登陆成功之后生成一个JWT令牌，并把这个了令牌直接返回给前端
2. 校验令牌：拦截前端请求，从请求中获取令牌，对令牌进行解析校验

```Java
package com.example.utils;  
  
import io.jsonwebtoken.Claims;  
import io.jsonwebtoken.Jwts;  
import io.jsonwebtoken.SignatureAlgorithm;  
  
import java.util.Date;  
import java.util.Map;  
  
public class JwtUtils {  
    private static String signKey = "example";//签名密钥  
    private static Long expire = 43200000L; //有效时间  
  
    /**  
     * 生成JWT令牌  
     * @param claims JWT第二部分负载payload中储存的内容  
     * @return  
     */  
    public static String generateJWT(Map<String, Object> claims) {  
        String jwt = Jwts.builder()  
                .addClaims(claims) // 自定义信息（有效载荷）  
                .signWith(SignatureAlgorithm.HS256, signKey) // 签名算法（头部）  
                .setExpiration(new Date(System.currentTimeMillis() + expire)) // 过期时间  
                .compact();  
        return jwt;  
    }  
  
    /**  
     * 解析JWT令牌  
     * @param jwt JWT令牌  
     * @return JTW第二部分负载payload中储存的内容  
     */  
    public static Claims parseJWT(String jwt) {  
        Claims claims = Jwts.parser()  
                .setSigningKey(signKey) // 指定签名密钥  
                .parseClaimsJws(jwt) // 指定令牌token  
                .getBody();  
        return claims;  
    }  
}
```

```Java
@RestController  
public class LoginController {  
    @Autowired  
    private EmpService empService;  
  
    // 登录  
    @PostMapping("/login")  
    public Result login(@RequestBody Emp emp) {  
        Emp loginEmp = empService.login(emp);  
  
        if (loginEmp != null) {  
            Map<String, Object> claims = new HashMap<>();  
            claims.put("id", loginEmp.getId());  
            claims.put("username", loginEmp.getUsername());  
            claims.put("name", loginEmp.getName());  
  
            // 使用JWT工具类，生成身份令牌  
            String token = JwtUtils.generateJWT(claims);  
            return Result.success(token);  
        }  
        return Result.error("用户名或密码错误");  
    }  
}
```
## 统一拦截技术
### 过滤器 Filter
- 过滤器可以把对资源的请求拦截下来，从而实现登录校验、统一编码处理、敏感字符处理等

#### 登录校验 Filter
- 除了登陆请求外，所有的请求都需要拦截并校验令牌
- 拦截到请求后，只有有令牌且令牌合法，才可以放行
![[filter-workflow.png]]
基于上面的业务流程，我们分析出具体的操作步骤：
1. 获取求url
2. 判断请求url中是否包含login，如果包含，说明是登录操作，放行
3. 获取请求头中的令牌（token）
4. 判断令牌是否存在，如果不存在，返回错误结果（未登录）
5. 解析token，如果解析失败，返回错误结果（未登录）
6. 放行

```Java
@Slf4j  
@WebFilter(urlPatterns = "/*")  
public class LoginCheckFilter implements Filter {  
    @Override  
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {  
        //前置：强制转换为http协议的请求对象、响应对象 （转换原因：要使用子类中特有方法）  
        HttpServletRequest request = (HttpServletRequest) servletRequest;  
        HttpServletResponse response = (HttpServletResponse) servletResponse;  
  
        //1.获取请求url  
        String url = request.getRequestURL().toString();  
        log.info("请求路径：{}", url); //请求路径：http://localhost:8080/login  
  
        //2.判断请求url中是否包含login，如果包含，说明是登录操作，放行  
        if (url.contains("/login")) {  
            chain.doFilter(request, response);//放行请求  
            return; //结束当前方法的执行  
        }  
  
        // 3. 获取请求头中的令牌  
        String token = request.getHeader("token");  
        log.info("从请求头中获取的令牌：{}", token);  
  
        // 4. 判断令牌是否存在，如果不存在，返回错误结果  
        if(!StringUtils.hasLength(token)) {  
            log.info("Token 不存在");  
  
            Result responseResult = Result.error("NOT_LOGIN");  
            //把Result对象转换为JSON格式字符串 (fastjson是阿里巴巴提供的用于实现对象和json的转换工具类)  
            String json = JSONObject.toJSONString(responseResult);  
            response.setContentType("application/json;charset=utf-8");  
            //响应  
            response.getWriter().write(json);  
            return;  
        }  
  
        // 5. 解析token，如果解析失败，返回错误结果（未登录）  
        try {  
            JwtUtils.parseJWT(token);  
        } catch (Exception e) {  
            log.info("令牌解析失败!");  
  
            Result responseResult = Result.error("NOT_LOGIN");  
            //手动把Result对象转换为JSON格式字符串返回前端 (fastjson是阿里巴巴提供的用于实现对象和json的转换工具类)  
            String json = JSONObject.toJSONString(responseResult);  
            // 设置相应头  
            response.setContentType("application/json;charset=utf-8");  
            //响应  
            response.getWriter().write(json);  
  
            return;  
        }  
  
        // 6. 放行  
        chain.doFilter(request, response);  
    }  
  
}
```

需添加依赖
```Java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.76</version>
</dependency>
```
### 拦截器 Interceptor
![[interceptor.png]]
拦截器与过滤器的区别：
- 接口规范不同：过滤器需要实现Filter接口，而拦截器需要实现HandlerInterceptor接口。
- 拦截范围不同：过滤器Filter会拦截所有的资源，而Interceptor只会拦截Spring环境中的资源。

#### 登录校验拦截器
```Java
//自定义拦截器
@Component //当前拦截器对象由Spring创建和管理
@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {
    //前置方式
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle .... ");
        //1.获取请求url
        //2.判断请求url中是否包含login，如果包含，说明是登录操作，放行

        //3.获取请求头中的令牌（token）
        String token = request.getHeader("token");
        log.info("从请求头中获取的令牌：{}",token);

        //4.判断令牌是否存在，如果不存在，返回错误结果（未登录）
        if(!StringUtils.hasLength(token)){
            log.info("Token不存在");

            //创建响应结果对象
            Result responseResult = Result.error("NOT_LOGIN");
            //把Result对象转换为JSON格式字符串 (fastjson是阿里巴巴提供的用于实现对象和json的转换工具类)
            String json = JSONObject.toJSONString(responseResult);
            //设置响应头（告知浏览器：响应的数据类型为json、响应的数据编码表为utf-8）
            response.setContentType("application/json;charset=utf-8");
            //响应
            response.getWriter().write(json);

            return false;//不放行
        }

        //5.解析token，如果解析失败，返回错误结果（未登录）
        try {
            JwtUtils.parseJWT(token);
        }catch (Exception e){
            log.info("令牌解析失败!");

            //创建响应结果对象
            Result responseResult = Result.error("NOT_LOGIN");
            //把Result对象转换为JSON格式字符串 (fastjson是阿里巴巴提供的用于实现对象和json的转换工具类)
            String json = JSONObject.toJSONString(responseResult);
            //设置响应头
            response.setContentType("application/json;charset=utf-8");
            //响应
            response.getWriter().write(json);

            return false;
        }

        //6.放行
        return true;
    }
```

在 `config/` 包中注册配置拦截器
```Java
@Configuration  
public class WebConfig implements WebMvcConfigurer {
    //拦截器对象
    @Autowired
    private LoginCheckInterceptor loginCheckInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
       //注册自定义拦截器对象
        registry.addInterceptor(loginCheckInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/login");
    }
}
```

# 全局异常处理
- 定义全局异常处理器：定义一个类，在类上加 `@RestControllerAdvice` 注解
- 在全局异常处理器中，我们需要：
	- 定义方法来捕获异常，加上注解 `@ExceptionHandler`，并通过其中的 `value` 属性来指定异常类型

在 `exception/` 包中：
```Java
@RestControllerAdvice
public class GlobalExceptionHandler {

    //处理异常
    @ExceptionHandler(Exception.class) //指定能够处理的异常类型
    public Result ex(Exception e){
        e.printStackTrace();//打印堆栈中的异常信息

        //捕获到异常之后，响应一个标准的Result
        return Result.error("对不起,操作失败,请联系管理员");
    }
}
```

@RestControllerAdvice = @ControllerAdvice + @ResponseBody
处理异常的方法返回值会转换为json后再响应给前端