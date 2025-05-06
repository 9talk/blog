# Spring Boot 多环境配置

## 使用yml文件

1. 使用多个配置文件, 命名规则: `application-{profile}.yml`, 典型文件: `application-dev.yml`, `application-prod.yml`
2. 使用单个配置文件, 使用 `---` 来作为分隔符, 使用 `spring.config.activate.on-profile: xxx`
   来进行声明当前配置在哪些环境下生效(依赖于`--spring.profiles.active=xxx`), 典型文件: `bootstrap.yml`, 历史版本使用`spring.profiles: xxx`来声明当前配置在哪些环境下生效, 但是已不再推荐使用
3. 遵循合并规则

> bootstrap.yml

```YAML
server:
   port: 16654
spring:
   application:
      name: service-gateway-mer
   cloud:
      consul:
         host: test-consul.nstl-dev.com
         port: 8500
         discovery:
            service-name: ${spring.application.name}
            prefer-ip-address: true #表示注册时使用IP而不是hostname
            instance-id: ${spring.application.name}:${server.port}:${spring.cloud.client.ip-address}:${version}
            health-check-url: http://${spring.cloud.client.ip-address}:${server.port}/health
            ip-address: ${spring.cloud.client.ip-address}
            health-check-interval: 60s
      config:
         discovery:
            enabled: false
         enabled: false
---
# 测试环境
spring:
   config:
      activate:
         on-profile: test
   cloud:
      consul:
         host: test-consul.nstl-dev.com
         port: 8500
---
# 预发布
spring:
   config:
      activate:
         on-profile: prepare
   cloud:
      consul:
         host: test-product-consul.nstl-dev.com
         port: 8500
---
# 生产
spring:
   config:
      activate:
         on-profile: product
   cloud:
      consul:
         host: consul.nstl-dev.com
         port: 8500
```

## 使用properties文件

1. 使用多个配置文件, 命名规则: `application-{profile}.properties`, 典型文件: `application-dev.properties`, `application-prod.properties`
2. 遵循和并规则

## 调用多环境配置

1. 调用多环境配置指的是: 调用除通用配置(`application.yml`,  `application.properties`)以外的配置. 通用配置(`application.yml`,  `application.properties`)会默认加载
2. 多环境的配置都依赖`spring.profiles.active`进行指定, 但是使用方式有多种, 推荐在启动参数时声明:
   * 在通用配置(`application.yml`,  `application.properties`)中指定`spring.profiles.active: xxx`进行声明
   * 在启动时指定`--spring.profiles.active=xxx`进行声明, 完整命令如下: `java -jar xxx.jar --spring.profiles.active=xxx`


## yml与properties文件总结(合并规则阐述)

1. yaml 配置方式明显优于 properties, yaml 配置文件支持多行, 语法更简洁, 语法错误提示更详细
2. yml 和 properties 都能合并对象属性, 根据配置文件的优先级, 优先级高的配置会覆盖优先级低的配置, 这是因为 spring 在加载配置时会加载所有配置文件, 并使用使用的打平(打宽)的方式进行使用Binder进行加载
3. 在使用合并时, 注意map和list的区别, list被认为是单key属性, 会覆盖处理, map会做合并处理
4. 这里的map指的是集合,可以是js object,java bean,HashMap, 等多种, 但是注意, map的key不能重复, 如果重复, 会被覆盖

## Maven Profile(不推荐, 因为乱)

如果使用的是构建工具是Maven，也可以通过Maven的profile特性来实现多环境配置打包。

> pom.xml
```xml
<profiles>
        <!--开发环境-->
        <profile>
            <id>dev</id>
            <properties>
                <build.profile.id>dev</build.profile.id>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <!--测试环境-->
        <profile>
            <id>test</id>
            <properties>
                <build.profile.id>test</build.profile.id>
            </properties>
        </profile>
        <!--生产环境-->
        <profile>
            <id>prod</id>
            <properties>
                <build.profile.id>prod</build.profile.id>
            </properties>
        </profile>
    </profiles>

    <build>
        <finalName>${project.artifactId}</finalName>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources.${build.profile.id}</directory>
                <filtering>false</filtering>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <classifier>exec</classifier>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

通过执行命令来指定使用哪个profile
`mvn clean package -P ${profile} `