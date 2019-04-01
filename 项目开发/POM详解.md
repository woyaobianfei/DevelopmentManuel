# POM详解

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.flnet.ms</groupId>
        <artifactId>ms-parent</artifactId>
        <version>1.9-RELEASE</version>
    </parent>

    <artifactId>ms-demo</artifactId>
    <version>1.0.0-RELEASE</version>
    <packaging>jar</packaging>

    <name>MS Demo</name>
    <description>MS Demo project for Spring Boot</description>

    <dependencies>

    </dependencies>
</project>
```

## Part 1

```xml
<parent>
    <groupId>com.flnet.ms</groupId>
    <artifactId>ms-parent</artifactId>
    <version>1.9-RELEASE</version>
</parent>
```

继承 Parent Pom - `com.flnet.ms.ms-parent`，`version`选择最新版本。

详细了解 [ms-parent](#ms-parent)。

## Part 2

```xml
<artifactId>ms-demo</artifactId>
<version>1.0.0-RELEASE</version>
<packaging>jar</packaging>
```

`artifactId`一般命名为项目缩写，管理系统项目以`ms-`作为前缀，**强制小写**。

初始版本为`1.0.0`，bug 修复升级版本号第三位，小版本迭代升级版本号第二位，大版本迭代升级版本号第一位。

## Part 3

```xml
<name>MS Demo</name>
<description>MS Demo project for Spring Boot</description>
```

`name`项目的英文名，首字母大写。

`description`项目描述，**禁止多项目间来回复制不做修改**。

## Part 4

```xml
<dependencies>
    
</dependencies>
```

引入依赖模块，常用模块已在`ms-parent`引用，如无特殊需求，无需引用即可开发。

## ms-parent

[查看源码](https://gogs.tvflnet.com/flw_tv/SpringBoot-Maven-Parent/src/master/Flnet-ManageSystem/pom.xml)

`ms-parent` 已默认集成模块：

* `spring-boot-starter` - SpringBoot
* `spring-boot-starter-test` - Test 
* `spring-boot-starter-log4j2` - log4j2
* `spring-boot-devtools` - DevTools
* `spring-boot-admin-starter-client` - SpringBoot Admin
* `pagehelper-spring-boot-starter` - pagehelper 分页插件
* `nacos-config-spring-boot-starter` - Nacos 配置中心
* `mysql-connector-java` - Mysql
* `druid-spring-boot-starter` - Druid 连接池
* `soa-api` - SOA 服务
* `permission-client` - 权限系统
* `redis-utils` - Redis
* `web-utils` -  web、aop、fastjson、joda-time、slf4j-api 等常用Jar

通过自定义`profile`, `resource.delimiter` 定界符和`resources`，实现分环境打包

```xml
<properties>
    <resource.delimiter>@</resource.delimiter>
</properties>

<!-- 环境变量选择参数 -->
<profiles>
    <!--本地环境-->
    <profile>
        <id>local</id>
        <properties>
            <spring.profiles.active>local</spring.profiles.active>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <!--开发环境-->
    <profile>
        <id>dev</id>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
    </profile>
    <!--测试环境-->
    <profile>
        <id>test</id>
        <properties>
            <spring.profiles.active>test</spring.profiles.active>
        </properties>
    </profile>
    <!--生产环境-->
    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
        </properties>
    </profile>
</profiles>

<build>
    <resources>
        <!-- 加载src/main/resource下的yml、properties文件 -->
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>*.yml</include>
                <include>*.properties</include>
            </includes>
            <filtering>true</filtering>
        </resource>
        <!-- 加载src/main/resource/分环境文件夹下的所有文件 -->
        <resource>
            <directory>src/main/resources/${spring.profiles.active}</directory>
            <filtering>false</filtering>
        </resource>
        <!-- 加载src/main/java下的xml文件,即mybatis文件 -->
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
</build>
```

流程说明：

1. `mvn -Pdev` maven 编译选定 profile 参数`dev`，默认`local`。
2. dev 对应的 profile 配置得`spring.profiles.active = dev`。
3. 拷贝配置文件`src/main/resources/*.yml`、 `src/main/resources/*.properties`、 `src/main/resources/dev/*`、 `src/main/java/**/*.xml`。
4. 根据定界符 @ 替换`application.yml`中`spring.profiles.active` 的 `@spring.profiles.active@` 为 `dev`。

*有时 Idea 自动编译可能导致`@spring.profiles.active@`替换失败，可以查看`target/classes`确认，失败时可手动重新编译。*



















