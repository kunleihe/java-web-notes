- MyBatis是一款优秀的 **持久层** **框架**，用于简化JDBC的开发。
- 使用Mybatis操作数据库，就是在Mybatis中编写SQL查询代码，发送给数据库执行，数据库执行后返回结果。
- Mybatis会把数据库执行的查询结果，使用实体类封装起来（一行记录对应一个实体类对象）

# 结构
![[my-batis-file-structure.png]]

## 实体类
- 在 pojo 包下面创建一个实体类
```Java
@Data
public class User {
    private Integer id;   //id（主键）
    private String name;  //姓名
    private Short age;    //年龄
    private Short gender; //性别
    private String phone; //手机号
}
```

## Mapper 包及接口
- 在创建出来的springboot工程中，在引导类所在包下，在创建一个包 mapper。
- 在mapper包下创建一个接口 UserMapper ，这是一个持久层接口（Mybatis的持久层接口规范一般都叫 XxxMapper）

### 编写SQL语句
```Java
import com.itheima.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import java.util.List;

@Mapper
public interface UserMapper {
    //查询所有用户数据
    @Select("select id, name, age, gender, phone from user")
    public List<User> list();
    
}
```

- @Mapper 注解：表示是 mybatis 中的 Mapper 接口
- 程序运行时，**框架会自动生成接口的实现类对象**，并交给 Spring 的 IOC 容器
- @Select 注解：代表的就是 select 查询，用于书写 select 查询语句
# 单元测试

```Java
@SpringBootTest
public class MybatisQuickstartApplicationTests {
    @Autowired
    private UserMapper userMapper;
    @Test
    public void testList(){
        List<User> userList = userMapper.list();
        for (User user : userList) {
            System.out.println(user);
        }
    }

}
```

# 数据库连接池
- 没有使用数据库连接池：
	- 客户端执行SQL语句：要先创建一个新的连接对象，然后执行SQL语句，SQL语句执行后又需要关闭连接对象从而释放资源，每次执行SQL时都需要创建连接、销毁链接，这种频繁的重复创建销毁的过程是比较耗费计算机的性能。
- 数据库连接池是个容器，负责分配、管理数据库连接(Connection)
	- 程序在启动时，会在数据库连接池(容器)中，创建**一定数量的Connection对象**
	- 允许应用程序**重复使用**一个现有的数据库连接，而不是再重新建立一个
	- 客户端在执行SQL时，先从连接池中获取一个Connection对象，然后在执行SQL语句，SQL语句执行完之后，释放Connection时就会把Connection对象归还给连接池（Connection对象可以复用）
	- 连接池会**释放空闲时间超过最大空闲时间的连接**，来避免因为没有释放连接而引起的数据库连接遗漏

数据库连接池的好处：
- 资源重用
- 提升系统响应速度
- 避免数据库连接遗漏
## 实现
- 官方(sun)提供了数据库连接池标准（javax.sql.DataSource接口）
- 常见的数据库连接池
	- C3P0, DBCP, Druid, **Hikari (Springboot 默认)**

# lombok
- Lombok是一个实用的Java类库，可以通过简单的注解来简化和消除一些必须有但显得很臃肿的Java代码。
- Lombok可以通过注解的形式自动生成构造器, getter/setter, equals, hashcode, toString等方法，并可以自动化日志变量，简化开发

|**注解**|**作用说明**|
|---|---|
|**`@Getter` / `@Setter`**|自动为类中所有的属性提供 `get` 和 `set` 方法。|
|**`@ToString`**|自动生成易阅读的 `toString` 方法，输出格式如 `ClassName(field1=value1, ...)`。|
|**`@EqualsAndHashCode`**|根据类中非静态（non-static）字段自动重写 `equals` 和 `hashCode` 方法。|
|**`@Data`**|**综合注解**。相当于同时添加了 `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode` + `@RequiredArgsConstructor`。|
|**`@NoArgsConstructor`**|为实体类生成一个**无参构造器**。|
|**`@AllArgsConstructor`**|为实体类生成一个**全参构造器**（包含所有字段，不包括 `static` 修饰的字段）。|
## 使用
1. 在 `pom.xml` 中倒入依赖
```Java
<!-- 在springboot的父工程中，已经集成了lombok并指定了版本号，故当前引入依赖时不需要指定version -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

2. 在实体类上添加注解
	- @Data //getter方法、setter方法、toString方法、hashCode方法、equals方法
	- @NoArgsConstructor //无参构造 
	- @AllArgsConstructor//全参构造
```Java
import lombok.Data;

@Data //getter方法、setter方法、toString方法、hashCode方法、equals方法 @NoArgsConstructor //无参构造 
@AllArgsConstructor//全参构造
public class User {
    private Integer id;
    private String name;
    private Short age;
    private Short gender;
    private String phone;
}
```