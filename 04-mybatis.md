- MyBatis是一款优秀的 **持久层** **框架**，用于简化JDBC的开发。
- 使用Mybatis操作数据库，就是在Mybatis中编写SQL查询代码，发送给数据库执行，数据库执行后返回结果。
- Mybatis会把数据库执行的查询结果，使用实体类封装起来（一行记录对应一个实体类对象）

# 结构
![[my-batis-file-structure.png]]

## 配置
在 resources/application.properties 中配置:

```yml
#驱动类名称
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#数据库连接的url
spring.datasource.url=jdbc:mysql://localhost:3306/database_name
#连接数据库的用户名
spring.datasource.username=root
#连接数据库的密码
spring.datasource.password=root
#指定mybatis输出日志的位置, 输出控制台  
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl 
#开启驼峰命名  
mybatis.configuration.map-underscore-to-camel-case=true
```

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

| **注解**                    | **作用说明**                                                                                                   |
| ------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **`@Getter` / `@Setter`** | 自动为类中所有的属性提供 `get` 和 `set` 方法。                                                                             |
| **`@ToString`**           | 自动生成易阅读的 `toString` 方法，输出格式如 `ClassName(field1=value1, ...)`。                                              |
| **`@EqualsAndHashCode`**  | 根据类中非静态（non-static）字段自动重写 `equals` 和 `hashCode` 方法。                                                        |
| **`@Data`**               | **综合注解**。相当于同时添加了 `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode` + `@RequiredArgsConstructor`。 |
| **`@NoArgsConstructor`**  | 为实体类生成一个**无参构造器**。                                                                                         |
| **`@AllArgsConstructor`** | 为实体类生成一个**全参构造器**（包含所有字段，不包括 `static` 修饰的字段）。                                                              |
## 使用
1. 在 `pom.xml` 中倒入依赖
```xml
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

@Data //getter方法、setter方法、toString方法、hashCode方法、equals方法 
@NoArgsConstructor //无参构造 
@AllArgsConstructor//全参构造
public class User {
    private Integer id;
    private String name;
    private Short age;
    private Short gender;
    private String phone;
}
```

# Mybatis 基础操作
## 日志输出

在 `application.properties 中输入：
```yml
#指定mybatis输出日志的位置, 输出控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

## 预编译
- Lombok日志中输出的SQL语句：`delete from emp where id = ?`，我们输入的参数 `16` 并没有在后面拼接，id的值是使用 `?` 进行占位。那这种 SQL 语句我们称为预编译 SQL
- 预编译的两个优势
	- 性能更高：预编译SQL，编译一次之后会将编译后的SQL语句缓存起来，后面再次执行这条语句时，不会再次编译。（只是输入的参数不同）
	- 更安全（防止 SQL 注入）
### SQL 注入
- 通过操作输入的数据来修改事先定义好的SQL语句，以达到执行代码对服务器进行攻击的方法。由于没有对用户输入进行充分检查，而SQL又是拼接而成，在用户输入参数时，在参数中添加一些SQL关键字，达到改变SQL运行结果的目的，也可以完成恶意攻击。
- 例如，在登陆界面，输入账号为 testtest，密码为 `' or '1' = '1'`, SQL query 变成了 `SELECT COUNT(*) FROM emp WHERE username = 'testtest' AND password = '' or '1' = '1';` 从而顺利登陆
![[sql-injection.png]]

## 参数占位符
在Mybatis中提供的参数占位符有两种：`${...}` 、`#{...}`
- `#{...}` - 建议使用
	- 执行SQL时，会将`#{…}`替换为`?`，生成预编译SQL，会自动设置参数值
	- 使用时机：参数传递，都使用`#{…}`
- `${...}`
	- 拼接SQL。直接将参数拼接在SQL语句中，存在SQL注入问题
	- 使用时机：如果对表名、列表进行动态设置时使用
## 数据封装
- 实体类属性名和数据库表查询返回的字段名一致，mybatis会自动封装。
- 如果实体类属性名和数据库表查询返回的字段名不一致，不能自动封装。
- 解决方案：

1. 在 SQL 语句中起别名
```SQL
@Select("select id, username, password, name, gender, image, job, entrydate, " +
        "dept_id AS deptId, create_time AS createTime, update_time AS updateTime " +
        "from emp " +
        "where id=#{id}")
public Emp getById(Integer id);
```
	
2. 手动结果映射
```SQL
@Results({@Result(column = "dept_id", property = "deptId"),
          @Result(column = "create_time", property = "createTime"),
          @Result(column = "update_time", property = "updateTime")})
@Select("select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp where id=#{id}")
public Emp getById(Integer id);
```

3. 开启驼峰命名（推荐）
在application.properties中添加：
```yml
mybatis.configuration.map-underscore-to-camel-case=true
```
要使用驼峰命名前提是 实体类的属性 与 数据库表中的字段名严格遵守驼峰命名。

### 模糊查询
- 使用MySQL提供的字符串拼接函数：`concat('%' , '关键字' , '%')`
```Java
@Mapper
public interface EmpMapper {

    @Select("select * from emp " +
            "where name like concat('%',#{name},'%') " +
            "and gender = #{gender} " +
            "and entrydate between #{begin} and #{end} " +
            "order by update_time desc")
    public List<Emp> list(String name, Short gender, LocalDate begin, LocalDate end);

}
```

# Mybatis 的 XML 配置文件
Mybatis的注解主要是来完成一些简单的增删改查功能。如果需要实现复杂的SQL功能，建议使用XML来配置映射语句，也就是将SQL语句写在XML配置文件中。

在Mybatis中使用XML映射文件方式开发，需要符合一定的规范：
1. XML映射文件的名称与Mapper接口名称一致，并且将XML映射文件和Mapper接口放置在相同包下（同包同名）
2. XML映射文件的namespace属性为Mapper接口全限定名一致
3. XML映射文件中sql语句的id与Mapper接口中的方法名一致，并保持返回类型一致。

## 第一步：创建 XML 映射文件
![[create-mybatis-xml.png]]

## 第二步：编写 XML 映射文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="复制mapper的reference path">
 
</mapper>
```

### 参数
- SQL方法的tag，如 `<select>`, `<insert>`, `<update>`, `<delete>`
- `id`: mapper里方法的名字
- `select` 需要有 `resultType` (返回一个实体类)
- `insert` `update` `delete` 需要有 `parameterType` （方法需要的参数）

案例
![[mybatis-xml-sql.png]]

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.example.mapper.EmpMapper">  
  
    <!--查询操作-->  
    <select id="list" resultType="com.example.pojo.Emp">  
        select * from emp  
        where name like concat('%',#{name},'%')  
        and gender = #{gender}  
        and entrydate between #{begin} and #{end}  
        order by update_time desc  
    </select>  
    </mapper>
```



# 动态 SQL
- 在页面原型中，列表上方的条件是动态的，是可以不传递的，也可以只传递其中的1个或者2个或者全部
- 而在我们刚才编写的SQL语句中，我们会看到，我们将三个条件直接写死了。 如果页面只传递了参数姓名name 字段，其他两个字段 性别 和 入职时间没有传递，那么这两个参数的值就是null。
- 此时，这个查询结果是不正确的。
- 正确的做法应该是：传递了参数，再组装这个查询条件；如果没有传递参数，就不应该组装这个查询条件。

## if
```SQL
<if test="条件表达式">
   要拼接的sql语句
</if>
```

原有的 SQL 语句
```SQL
<select id="list" resultType="com.example.pojo.Emp">
        select * from emp
        where name like concat('%',#{name},'%')
              and gender = #{gender}
              and entrydate between #{begin} and #{end}
        order by update_time desc
</select>
```

动态 SQL 语句
```SQL
<select id="list" resultType="com.example.pojo.Emp">
        select * from emp
        where
             <if test="name != null">
                 name like concat('%',#{name},'%')
             </if>
             <if test="gender != null">
                 and gender = #{gender}
             </if>
             <if test="begin != null and end != null">
                 and entrydate between #{begin} and #{end}
             </if>
        order by update_time desc
</select>
```

> [!tip] 如果姓名为 null 会报错!
> 当你传入的 `name` 为 `null` 时，第一个 `<if>` 块会被跳过，生成的 SQL 会变成：
> ```SQL
> select * from emp
> where 
> -- 这里由于 name 为 null，漏掉了条件
 > and gender = ? 
  > and entrydate between ? and ?
> order by update_time desc
> ```
> 此时，`where` 关键字后面直接紧跟了一个 `and`，这在 SQL 语法中是非法的，导致数据库执行失败。

## <>
所以，需使用`<where>`标签代替SQL语句中的`where`关键字:
- `<where>`只会在子元素有内容的情况下才插入where子句，而且会自动去除子句的开头的AND或OR

```SQL
<select id="list" resultType="com.example.pojo.Emp">
        select * from emp
        <where>
             <!-- if做为where标签的子元素 -->
             <if test="name != null">
                 and name like concat('%',#{name},'%')
             </if>
             <if test="gender != null">
                 and gender = #{gender}
             </if>
             <if test="begin != null and end != null">
                 and entrydate between #{begin} and #{end}
             </if>
        </where>
        order by update_time desc
</select>
```

同理，在更新操作时，使用`<set>` 
动态的在SQL语句中插入set关键字，并会删掉额外的逗号。

## foreach
```xml
<foreach collection="集合名称" item="集合遍历出来的元素/项" separator="每一次遍历使用的分隔符"
         open="遍历开始前拼接的片段" close="遍历结束后拼接的片段">
</foreach>
```
![[foreach.png]]
## sql&include
- 在xml映射文件中配置的SQL，有时可能会存在很多重复的片段，此时就会存在很多冗余的代码
- 我们可以对重复的代码片段进行抽取，将其通过`<sql>`标签封装到一个SQL片段，然后再通过`<include>`标签进行引用。
	- `<sql>`：定义可重用的SQL片段
	- `<include>`：通过属性refid，指定包含的SQL片段

SQL片段： 抽取重复的代码
```SQL
<sql id="commonSelect">
    select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp
</sql>
```

然后通过`<include>` 标签在原来抽取的地方进行引用。操作如下：
```SQL
<select id="list" resultType="com.itheima.pojo.Emp">
    <include refid="commonSelect"/>
    <where>
        <if test="name != null">
            name like concat('%',#{name},'%')
        </if>
        <if test="gender != null">
            and gender = #{gender}
        </if>
        <if test="begin != null and end != null">
            and entrydate between #{begin} and #{end}
        </if>
    </where>
    order by update_time desc
</select>
```
 
