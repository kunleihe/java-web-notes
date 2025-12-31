# 配置优先级
在SpringBoot项目当中，常见的属性配置方式有5种， 3种配置文件，加上2种外部属性的配置(Java系统属性、命令行参数)。通过以上的测试，我们也得出了优先级(从高到低)：
- 命令行参数（--xxx=xxx）
- Java系统属性
- application.properties
- application.yml
- application.yaml

命令行参数和Java系统属性在运行程序 --> Edit configuration中添加
- 系统属性参数在 VM options 中添加
- 命令行参数在 Program arguments 中添加