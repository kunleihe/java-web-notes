- Maven是Apache旗下的一个开源项目，是一款用于管理和构建java项目的工具.
- Maven的是三个作用：
	- 管理依赖
	- 统一项目结构
	- 项目构建
- Maven提供了跨平台的自动化构建方式（清理 - 编译 - 测试 - 打包 - 发布）
![[maven-proj-structure.png]]
## Maven坐标
- Maven中的坐标是==资源的唯一标识== , 通过该坐标可以唯一定位资源位置
- 使用坐标来定义项目或引入项目中需要的依赖

Maven坐标主要组成
- groupId：定义当前Maven项目隶属组织名称（通常是域名反写，例如：com.itheima）
- artifactId：定义当前Maven项目名称（通常是模块名称，例如 order-service、goods-service）
- version：定义当前项目版本号
![[Screenshot 2025-12-19 at 10.09.52 PM.png]]

## 依赖管理
依赖：指当前项目运行所需要的jar包。一个项目中可以引入多个依赖：
```HTML
<dependencies>
    <!-- 第1个依赖 : logback -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.11</version>
    </dependency>
    <!-- 第2个依赖 : junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```
- 如果引入的依赖，在本地仓库中不存在，将会连接远程仓库 / 中央仓库，然后下载依赖（这个过程会比较耗时，耐心等待）
- 如果不知道依赖的坐标信息，可以到mvn的中央仓库（[https://mvnrepository.com/](https://mvnrepository.com/)）中搜索
### 依赖传递
- 依赖具有传递性，当我们导入一个依赖时，它所需要的依赖会被自动导入

#### 直接依赖 vs 间接依赖
![[maven-dependency.png]]
- 对于 Project A 来说， Project B 是直接依赖
- Project A 来说，Project C 是间接依赖
#### 排除依赖
- 指主动断开依赖的资源。（被排除的资源无需指定版本）

```HTML
<dependency>
    <groupId>com.itheima</groupId>
    <artifactId>maven-projectB</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--排除依赖, 主动断开依赖的资源-->
    <exclusions>
        <exclusion>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### 依赖范围
- 在项目中导入依赖的jar包后，默认情况下，可以在任何地方使用。
- 如果希望限制依赖的使用范围，可以通过`<scope>`标签设置其作用范围。
- scope的取值范围
	- compile (default)
	- test
	- provided
	- runtime

```HTML
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <!-- 仅测试环境有效 -->
    <scope>test</scope> 
</dependency>
```
### 生命周期
clean --> validate --> compile --> test --> package --> verify --> install --> site --> deploy
我们需要关注的就是：clean --> compile --> test --> package --> install