# Web 分析
### 浏览器：
- 输入网址 `http://192.168.100.11:8080/hello`
	- 通过IP地址192.168.100.11定位到网络上的一台计算机
		- 我们之前在浏览器中输入`localhost`，就是127.0.0.1（本机）
	- 通过端口号8080找到计算机上运行的程序
		- `localhost:8080` , 意思是在本地计算机中找到正在运行的8080端口的程序
	- `/hello`是请求资源位置
		- 资源：对于计算机而言就是数据
		- web资源：通过网络可以访问到的资源（通常指存放在服务器上的数据）
	- `localhost:8080/hello` ，意思是向本地计算机中的`8080`端口程序，获取资源位置是`/hello`的数据
### 服务器：
- 接收到浏览器发送的信息（如：/hello）
- 在服务器上找到/hello的资源
- 把资源发送给浏览器
### 网络三要素
- IP ：网络中计算机的唯一标识
- 端口 ：计算机中运行程序的唯一标识
- 协议 ：网络中计算机之间交互的规则
# HTTP协议
## 特点
- 基于TCP
- 基于请求-响应模型
- 无状态
## HTTP-请求协议
- 请求协议： 浏览器将数据以请求格式发送到服务器
	- 包括：请求行 (line)、请求头 (head)、请求体 (body)
- 响应协议：服务器将数据以响应格式返回给浏览器
	- 包括：**响应行** 、**响应头** 、响应体
### 浏览器访问服务器的几种方式
- **GET**
- **POST**
- OPTIONS
- HEAD
- PUT
- DELETE
- TRACE
- CONNECT
#### GET 方式的请求协议
![[get.png]]
- 请求行 = 请求方式 + 资源路径 + 协议/版本 （之间使用空格分隔）
	- 请求方式 GET
	- 资源路径：/brand/findAll?name=OPPO$status=1
		- 请求路径：/brand/findAll
		- 请求参数：name=OPPO&status=1
			- 请求参数是以 key=value 形式出现
			- 多个请求参数之间用 & 连接
			- 请求路径与请求参数之间用 ? 连接
	- 协议/版本：HTTP/1.1
- 请求头
	- 格式为 key:value
	- http是个无状态的协议，所以在请求头设置浏览器的一些自身信息和想要响应的形式。这样服务器在收到信息后，就可以知道是谁，想干什么了
	- 常见的HTTP请求头
```HTML
Host: 表示请求的主机名

User-Agent: 浏览器版本。 例如：Chrome浏览器的标识类似Mozilla/5.0 ...Chrome/79 ，IE浏览器的标识类似Mozilla/5.0 (Windows NT ...)like Gecko

Accept：表示浏览器能接收的资源类型，如text/*，image/*或者*/*表示所有；

Accept-Language：表示浏览器偏好的语言，服务器可以据此返回不同语言的网页；

Accept-Encoding：表示浏览器可以支持的压缩类型，例如gzip, deflate等。

Content-Type：请求主体的数据类型

Content-Length：数据主体的大小（单位：字节）
```

- 请求体：储存请求参数
	- GET 请求的请求参数在请求行中，故不需要设置请求体
#### POST 方式的请求协议
![[post.png]]
- 请求行(以上图中红色部分)：包含请求方式、资源路径、协议/版本
    - 请求方式：POST
    - 资源路径：/brand
    - 协议/版本：HTTP/1.1
- 请求头(以上图中黄色部分)
- 请求体(以上图中绿色部分) ：存储请求参数 (与 GET 不同)
## HTTP-响应协议
![[response.png]]
- 响应行：响应数据的第一行。由协议、版本、响应状态码、状态码描述组成
	- 协议/版本：HTTP/1.1
	- 响应状态码：200
	- 状态码描述：OK
- 响应头：黄色部分。格式为 key:value
- 响应体：储存响应的数据
### 响应状态码

| 状态码分类 | 说明                                                     |
| ----- | ------------------------------------------------------ |
| 1xx   | 响应中 --- 临时状态码。表示请求已经接受，告诉客户端应该继续请求或者如果已经完成则忽略          |
| 2xx   | 成功 --- 表示请求已经被成功接收，处理已完成                               |
| 3xx   | 重定向 --- 重定向到其它地方，让客户端再发起一个请求以完成整个处理                    |
| 4xx   | 客户端错误 --- 处理发生错误，责任在客户端，如：客户端的请求一个不存在的资源，客户端未被授权，禁止访问等 |
| 5xx   | 服务器端错误 --- 处理发生错误，责任在服务端，如：服务端抛出异常，路由出错，HTTP版本不支持等     |
# WEB 服务器 - Tomcat
- 服务器只是一台设备，必须安装服务器软件才能提供相应的服务。
- 服务器软件：基于ServerSocket编写的程序
	- 服务器软件本质是一个运行在服务器设备上的应用程序
	- 能够接收客户端请求，并根据请求给客户端响应数据
## 入门程序解析
### 起步依赖
- SpringBoot的项目中，有很多的起步依赖，他们有一个共同的特征：就是以`spring-boot-starter-`作为开头
- spring-boot-starter-web：包含了web应用开发所需要的常见依赖
- spring-boot-starter-test：包含了单元测试所需要的常见依赖
# Browser-Server 架构
![[bs.png]]
- BS架构：Browser/Server，浏览器/服务器架构模式。客户端只需要浏览器，应用程序的逻辑和数据都存储在服务端。
## 请求
### 简单参数
```Java
@RestController
public class RequestController {
    // http://localhost:8080/simpleParam?name=Tom&age=10
    // 第1个请求参数： name=Tom   参数名:name，参数值:Tom
    // 第2个请求参数： age=10     参数名:age , 参数值:10
    //springboot方式
    @RequestMapping("/simpleParam")
    public String simpleParam(String name , Integer age ){//形参名和请求参数名保持一致
        System.out.println(name+"  :  "+age);
        return "OK";
    }
}
```

如果方法形参名称与请求参数名称不一致怎么办？
- 返回 null

解决办法：
- 在方法形参前面加上 `@RequestParam` 然后通过`value`属性执行请求参数名，从而完成映射
```Java
@RestController
public class RequestController {
    // http://localhost:8080/simpleParam?name=Tom&age=20
    // 请求参数名：name

    //springboot方式
    @RequestMapping("/simpleParam")
    public String simpleParam(@RequestParam("name") String username , Integer age ){
        System.out.println(username+"  :  "+age);
        return "OK";
    }
}
```

> [!info] @RequestParam 中的 required 属性
> 默认为true（默认值也是true），代表该请求参数必须传递，如果不传递将报错。
> 如果该参数是可选的，可以将 required 属性设置为 false  
> 

```Java
@RequestMapping("/simpleParam")
public String simpleParam(@RequestParam(name = "name", required = false) String username, Integer age){
 System.out.println(username+ ":" + age);
 return "OK";
}
```
### 实体参数
- 使用简单参数来传递数据时，前端传递了多少请求参数，后端 controller 方法中的形参就要写几个。如果请求参数比较多，会比较麻烦
- 此时，我们可以考虑把请求参数封装到一个实体类对象中。
- 需要遵守如下规则：请求参数名与实体类属性名相同
	- 请求参数名与实体类属性名不相同时，会返回 null
#### 简单实体对象
```Java
public class User {
    private String name;
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

Controller 方法

```java
@RestController
public class RequestController {
	// 实体参数：简单实体对象
	@RequestMapping("/simpleParam")
	public String simplePojo(User user) {
		System.out.println(user);
		return "OK";
	}
}
```
#### 复杂实体对象
- 复杂实体对象指的是，在实体类中有一个或多个属性，也是实体对象类型的。
- 请求参数名与形参对象属性名相同，按照对象层次结构关系即可接收嵌套实体类属性参数
#### 数组集合参数
- 在HTML表单中，有些表单支持多选，多个值是一个一个提交的
- 后端程序接受上述多个值有两种方式
	- 数组
	- 集合
##### 数组
- 请求参数名与形参数组名称相同且请求参数为多个，定义数组类型即可接收参数
![[array.png]]
##### 集合
- 请求参数名与形参集合对象名相同且请求参数为多个，@RequestParam 绑定参数关系
![[list.png]]
### 日期参数
- 因为日期的格式多种多样，对于日期类型的参数进行封装时，需通过 `@DateTimeFormat` 注解，以及其 pattern 属性来设置日期的格式
![[date.png]]
### JSON 参数
- Postman 发送 JSON 格式数据，用 Body -> raw (JSON)
- 服务端 Controller 接受 JSON格式数据
	- 用实体类进行封装
	- 封装规则：
		- JSON 数据键名与形参对象属姓名相同，定义 POJO 类型形参即可接收参数，需要使用 `@RequestBody`标识
		- `@RequestBody` 注解：将JSON数据映射到形参的实体类对象中（JSON 中的 key 和实体类中的属性名保持一致）
```java
@RestController
public class RequestController {
	@RequestMapping("/jsonParam")
	public String jsonParam(@RequestBody User user) {
	System.out.println(user);
	return "OK";
	}
}
```
### 路径参数
- 前端：通过请求 URL 直接传递参数
- 后端： 使用 {...} 来标识路径参数

```java
@RestController
public class RequestController {
	// 路径参数
	@RequestMapping("/path/{id}")
	public String pathParam(@PathVariable Integer id) {
		System.out.println(id);
		return "OK";
	}
}
```
#### 传递多个路径参数
```java
@RestController
public class RequestController {
	@RequestMapping("/path/{id}/{name}")
	public String pathParam2(@PathVariable Integer id, @PathVariable String name) {
		System.out.println(id + " : " + name);
		return "OK";
	}
}
```
## 响应
- controller 方法中 return 的结果，可以通过 @RequestBody 注解响应给浏览器
- 我们常用的 @RequestController = @Controller + @RequestBody
- 方法的返回值如果是一个 POJO 对象或集合，会先转换为 JSON 格式，再响应给浏览器
```java
@RestController
public class ResponseController{
	// 响应字符串
	@RequestMapping("/hello") {
		System.out.println("Hello World!");
		return "Hello World!";
	}
	
	// 响应实体对象
	@RequestMapping("/getAddr")
		public Address getAddr() {
			Address addr = new Address(); // 创建实体类对象
			addr.setProvince("广东");
			addr.setCity("深圳");
			return addr;
		}
	
	// 响应集合数据
	@RequestMapping("/listAddr")
	public List<Address> listAddr() {
		List<Address> list = new ArrayList<>();
		Address addr = new Address();
		addr.setProvince("广东");
		addr.setCity("深圳");
		
		Address addr2 = new Address();
		addr2.setProvince("山西");
		addr2.setCity("太原");		
		
		list.add(addr);
		list.add(addr2);
		return list;
	}
}
```
### 实体类 Result
- 定义一个统一的返回结果
- 统一的返回结果使用类来描述，在这个结果中包含：
	- 响应状态码：当前请求是成功，还是失败
	- 状态码信息：给页面的提示信息
	- 返回的数据：给前端响应的数据（字符串、对象、集合）

```Java
public class Result {
    private Integer code;//响应码，1 代表成功; 0 代表失败
    private String msg;  //响应码 描述字符串
    private Object data; //返回的数据

    public Result() { }
    public Result(Integer code, String msg, Object data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }

    //增删改 成功响应(不需要给前端返回数据)
    public static Result success(){
        return new Result(1,"success",null);
    }
    //查询 成功响应(把查询结果做为返回数据响应给前端)
    public static Result success(Object data){
        return new Result(1,"success",data);
    }
    //失败响应
    public static Result error(String msg){
        return new Result(0,msg,null);
    }
}
```

改造 Controller
```Java
@RestController
public class ResponseController {
    //响应统一格式的结果
    @RequestMapping("/hello")
    public Result hello(){
        System.out.println("Hello World ~");
        //return new Result(1,"success","Hello World ~");
        return Result.success("Hello World ~");
    }

    //响应统一格式的结果
    @RequestMapping("/getAddr")
    public Result getAddr(){
        Address addr = new Address();
        addr.setProvince("广东");
        addr.setCity("深圳");
        return Result.success(addr);
    }

    //响应统一格式的结果
    @RequestMapping("/listAddr")
    public Result listAddr(){
        List<Address> list = new ArrayList<>();

        Address addr = new Address();
        addr.setProvince("广东");
        addr.setCity("深圳");

        Address addr2 = new Address();
        addr2.setProvince("陕西");
        addr2.setCity("西安");

        list.add(addr);
        list.add(addr2);
        return Result.success(list);
    }
}
```
## 分层解耦
### 分层
- Controller: 控制层。接收前端发送的请求，对请求进行处理，并响应数据
- Service: 业务逻辑层。处理具体的业务逻辑。
- DAO: 数据访问层 (Data Access Object) 也称为持久层。负责数据访问操作，包括增删改查。
![[three-layer.png]]
### 解耦
- 之前我们在编写代码时，需要什么对象，直接 new 一个。但是这样做，层与层之间就耦合了。当 service 层的实现变了之后，我们还需要修改 controller 层的代码
![[coupling.png]]
解耦思路
- 不能 new
- 提供一个容器，容器中储存一些对象
- controller 程序从容器中获取 `EmpService` 类型的对象

Spring中的两个核心概念
- 控制反转 (Inversion of Control, IOC)
	- 对象的创建控制权由程序自身转移到容器。这个容器成为 IOC 容器、或 Spring 容器
- 依赖注入 (Dependency Injection, DI)
	- 容器为应用程序提供运行时所依赖的资源
- IOC 容器中创建、管理的对象叫 Bean对象
#### IOC 详解
@Component 衍生注解
- @Controller 标注在控制层类上
- @Service 标注在业务层类上
- @Repository 标注在数据访问层类上

Bean 的名字
- 默认名字：类名首字母小写
- 声明 bean 的时候，可以通过 value 属性制定 bean 的名字

组件扫描
- 将我们定义的controller，service，dao这些包都放在引导类所在包com.itheima的子包下，这样我们定义的bean就会被自动的扫描到

#### DI 详解
@Autowired 注解
- 自动装配
- 如果在IOC容器中，存在多个相同类型的bean对象，会报错

解决方案
- @Primary
	- 当存在多个相同类型的Bean注入时，加上@Primary注解，来确定默认的实现
![[Screenshot 2025-12-21 at 10.34.41 PM.png]]
- @Qualifier
	- 指定当前要注入的bean对象。 在@Qualifier的value属性中，指定注入的bean的名称。
	- @Qualifier注解不能单独使用，必须配合@Autowired使用
![[qualifier.png]]
- @Resource
	- 是按照bean的名称进行注入。通过name属性指定要注入的bean的名称。
![[resource.png]]