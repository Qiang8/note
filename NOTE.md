# note
spring boot的应用
首先创建一个一般的Maven项目，有一个pom.xml和基本的src/main/java结构
pom.xml文件的配置如下:
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.github.abel533</groupId>
    <artifactId>spring-boot</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.0.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <dependencies>
                    <dependency>
                        <groupId>org.springframework</groupId>
                        <artifactId>springloaded</artifactId>
                        <version>1.2.5.RELEASE</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>

首先是增加了<parent>

增加父pom比较简单，而且spring-boot-starter-parent包含了大量配置好的依赖管理，在自己项目添加这些依赖的时候不需要写<version>版本号。

使用父pom虽然简单，但是有些情况我们已经有父pom，不能直接增加<parent>时，可以通过如下方式：

<dependencyManagement>
     <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.2.3.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

java.version属性

上面pom.xml虽然没有出现这个属性，这里要特别提醒。

Spring默认使用jdk1.6，如果想使用jdk1.8，需要在pom.xml的属性里面添加java.version，如下：

<properties>
    <java.version>1.8</java.version>
</properties>

添加spring-boot-starter-web依赖

Spring通过添加spring-boot-starter-*这样的依赖就能支持具体的某个功能。

我们这个示例最终是要实现web功能，所以添加的是这个依赖。

更完整的功能列表可以查看：using-boot-starter-poms

添加spring-boot-maven-plugin插件

该插件支持多种功能，常用的有两种，第一种是打包项目为可执行的jar包。

在项目根目录下执行mvn package将会生成一个可执行的jar包，jar包中包含了所有依赖的jar包，只需要这一个jar包就可以运行程序，使用起来很方便。该命令执行后还会保留一个XXX.jar.original的jar包，包含了项目中单独的部分。

生成这个可执行的jar包后，在命令行执行java -jar xxxx.jar即可启动项目。

另外一个命令就是mvn spring-boot:run，可以直接使用tomcat（默认）启动项目。

在我们开发过程中，我们需要经常修改，为了避免重复启动项目，我们可以启用热部署。

Spring-Loaded项目提供了强大的热部署功能，添加/删除/修改 方法/字段/接口/枚举 等代码的时候都可以热部署，速度很快，很方便。

想在Spring Boot中使用该功能非常简单，就是在spring-boot-maven-plugin插件下面添加依赖：

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>springloaded</artifactId>
        <version>1.2.5.RELEASE</version>
    </dependency>
</dependencies>

添加以后，通过mvn spring-boot:run启动就支持热部署了。

注意：使用热部署的时候，需要IDE编译类后才能生效，可以打开自动编译功能，这样在保存修改的时候，类就自动重新加载了。

创建一个应用类

我们创建一个Application类：

@RestController
@EnableAutoConfiguration
public class Application {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    @RequestMapping("/now")
    String hehe() {
        return "现在时间：" + (new Date()).toLocaleString();
    }

    public static void main(String[] args) {
        SpringApplication.run(Example.class, args);
    }

}


Spring Boot建议将我们main方法所在的这个主要的配置类配置在根包名下。


在Application.java中有main方法。

因为默认和包有关的注解，默认包名都是当前类所在的包，例如@ComponentScan, @EntityScan, @SpringBootApplication注解。

@RestController

因为我们例子是写一个web应用，因此写的这个注解，这个注解相当于同时添加@Controller和@ResponseBody注解。

@EnableAutoConfiguration

Spring Boot建议只有一个带有该注解的类。

@EnableAutoConfiguration作用：Spring Boot会自动根据jar包的依赖来自动配置项目。例如当项目下面有HSQLDB的依赖时，Spring Boot会创建默认的内存数据库的数据源DataSource，如果自己创建了DataSource，Spring Boot就不会创建默认的DataSource。

如果不想让Spring Boot自动创建，可以配置注解的exclude属性，例如：

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {

@SpringBootApplication

由于大量项目都会在主要的配置类上添加@Configuration,@EnableAutoConfiguration,@ComponentScan三个注解。

因此Spring Boot提供了@SpringBootApplication注解，该注解可以替代上面三个注解（使用Spring注解继承实现）。

home等方法

@RequestMapping("/")
String home() {
    return "Hello World!";
}

@RequestMapping("/now")
String hehe() {
    return "现在时间：" + (new Date()).toLocaleString();
}

这些方法都添加了@RequestMapping("xxx")，这个注解起到路由的作用。

启动项目SpringApplication.run

启动Spring Boot项目最简单的方法就是执行下面的方法：

SpringApplication.run(Application.class, args);
1
该方法返回一个ApplicationContext对象，使用注解的时候返回的具体类型是AnnotationConfigApplicationContext或AnnotationConfigEmbeddedWebApplicationContext，当支持web的时候是第二个。

除了上面这种方法外，还可以用下面的方法：

SpringApplication application = new SpringApplication(Application.class);
application.run(args);
1
2
SpringApplication包含了一些其他可以配置的方法，如果想做一些配置，可以用这种方式。

除了上面这种直接的方法外，还可以使用SpringApplicationBuilder：

new SpringApplicationBuilder()
        .showBanner(false)
        .sources(Application.class)
        .run(args);

当使用SpringMVC的时候由于需要使用子容器，就需要用到SpringApplicationBuilder，该类有一个child(xxx...)方法可以添加子容器。

运行

在IDE中直接直接执行main方法，然后访问http://localhost:8080即可。

另外还可以用上面提到的mvn，可以打包为可执行jar包，然后执行java -jar xxx.jar。

或者执行mvn spring-boot:run运行项目。
