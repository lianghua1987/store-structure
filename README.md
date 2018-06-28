# Hua - Store

An e-commerce store designed by Hua, and just for fun using some java8 technology, solr, redis cluster, nginx.

## Background

E-commerce website is popular in nowadays market, especially in China. In order to make myself competent, I need to learn how to build a e-commerce website from scrach.

## Agenda

| DAYS | Agenda                                                       | Finished | Date                                           |
| :--: | ------------------------------------------------------------ | :------: | ---------------------------------------------- |
|  1   | Background, history, now and future. Setup maven project, GitHub repo created. |    ✅     | 04/18/2018, 04/19/2018, 04/19/2018             |
|  2   | **Framework intergration.** Products list implemetation, paganation. |    ✅     | 04/21/2018, 04/22/2018, 04/23/2018             |
|  3   | Backend service management. Add product, image upload.       |    ✅     | 04/24/2018, 04/25/2018, 04/26/2018, 04/27/2018 |
|  4   | Product structure, info.                                     |    ✅     | 04/28/2018                                     |
|  5   | Product front end, display page.                             |    ✅     | 04/28/2018, 04/29/2018                         |
|  6   | cms implementation. Ad display.                              |    ✅     | 04/29/2018, 04/30/2018                         |
|  7   | **Add cache, Redis, cache synchornaztion.**                  |    ✅     | 04/30/2018, 05/01/2018, 05/02/2018, 05/03/2018 |
|  8*  | **Search function. Implement by solr.**                      |    ✅     | 05/03/2018, 05/04/2018, 05/05/2018, 05/06/2018 |
|  9   | Product detail page.                                         |    ✅     | 06/01/2018, 06/02/2018                         |
|  10  | **Shared session.**                                          |    ✅     | 06/03/2018                                     |
|  11  | Shopping cart.                                               |          |                                                |
|  12  | **Nginx.**                                                   |          |                                                |
|  13  | **Redis cluster, solr cluster. System deployment.**          |          |                                                |
|  14  | Wrap up                                                      |          |                                                |

## Features

- Backend: Product mangement, order managerment, category management, user management, publisher
- Frontend: User can register, login, broswer products, homepage and place an order
- Membership: Order history, bookmarked product, coupon code and groupon
- Order: Place, search, modify and order scheduler
- Seach: Prodct search function

## Tech Approach

- Spring, SpringMVC, Mybatis
- JSP, JSTL, jQuery, plugin, EasyUI, KindEditor
- Redis
- Solr
- httpClient
- Mysql
- Nginx
- Tomcat

## Maven 

| Pom Files     | Extension |
| ------------- | --------- |
| store-parent  | .pom      |
| store-common  | .jar      |
| store-manager | .pom      |

## Structure

store-parent

|— store-common

|— store-manager

​       |— com.hua.manager.web(.**war**)

​       |— com.hua.manager.service

​       |— com.hua.manager.mapper

​       |— com.hua.manager.pojo

Config:

```xml
// under store-manager, pom.xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <configuration>
                    <port>8080</port>
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

Run

```shell
// under store-manager
mvn clean install
mvn clean tomcat7:run
```

Framework: springMVC + spring + mybatis

### Database configuration, entity creation

1. create database: store
2. load init.sql(/scripts)
3. Reverse engineering -> db schema to java entity

**mvn clean install — plugin way**

```xml
<plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.5</version>
                <configuration>
                    	<configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                </configuration>
                <executions>
                    <execution>
                        <id>Generate MyBatis Artifacts</id>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.3.5</version>
                    </dependency>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>5.1.28</version>
                    </dependency>
                </dependencies>
            </plugin>
```

**** **java generator**

```java
public void generator() throws Exception{
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        File configFile = new File(SchemaToEntity.class.getClassLoader().getResource("generatorConfig.xml").getPath());

        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
                callback, warnings);
        myBatisGenerator.generate(null);
    }

    public static void main(String[] args) throws Exception {
        try {
            new SchemaToEntity().generator();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

[Introduction to MyBatis Generator]: http://www.mybatis.org/generator/index.html
[MyBatis Generator Release 1.3.6]: https://github.com/mybatis/generator/releases



### Integration SSM(SpringMVC, Spring, Mybatis)	

#### DAO Layer

Using mybatis framework, create *sqlMapConfig.xml*, *applicationContext-dao.xml*

1. Datasource configuration
2. SpingIoc manage *SqlSessionFactory*, singleton
3. Add mapper object to Springioc

#### Service Layer

1. Transaction management
2. Add service to SpringIoc

#### ViewModel Layer

1. Annotation configuration
2. ModelViewResolver configuration
3. Scan controller

#### Web.xml

1. SpringIoc configuration
2. SpringMvc configuration
3. Post encoding

#### Configurations - Under store-manager-web

##### *Config.xml*

```Xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <!--<property name="dialect" value="mysql"/>-->
        </plugin>
    </plugins>
```

##### *ApplicationContext-dao.xml*

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

    <!-- Configuration file-->
    <context:property-placeholder location="classpath:db.properties" />

    <!-- Connection pool - druid -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="maxActive" value="10" />
        <property name="minIdle" value="5" />
    </bean>

    <!-- Session Factory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">

        <property name="dataSource" ref="dataSource" />
        <!-- load mybatis global configuration -->
        <property name="configLocation" value="classpath:mybatis/config.xml" />
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.hua.store.mapper" />
    </bean>
</beans>
```

##### *ApplicationContext-service.xml*

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

    <context:component-scan base-package="com.hua.store.service"/>

</beans>
```

##### *ApplicationContext-transaction.xml*

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED" />
            <tx:method name="insert*" propagation="REQUIRED" />
            <tx:method name="add*" propagation="REQUIRED" />
            <tx:method name="create*" propagation="REQUIRED" />
            <tx:method name="delete*" propagation="REQUIRED" />
            <tx:method name="update*" propagation="REQUIRED" />
            <tx:method name="find*" propagation="SUPPORTS" read-only="true" />
            <tx:method name="select*" propagation="SUPPORTS" read-only="true" />
            <tx:method name="get*" propagation="SUPPORTS" read-only="true" />
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:advisor advice-ref="txAdvice"
                     pointcut="execution(* com.hua.store.service.*.*(..))" />
    </aop:config>
</beans>
```

##### *springmvc.xml*

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.hua.store.controller" />
    
    <mvc:annotation-driven />
    <bean
            class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>
    
    <!-- Static resources mapping -->
   <mvc:resources mapping="/css/**" location="/WEB-INF/css/"/>
   <mvc:resources mapping="/js/**" location="/WEB-INF/js/"/>
</beans>
```

##### *web.xml*

```xml
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>

    <display-name>store-manager-web</display-name>
    <welcome-file-list>
        <welcome-file>login.html</welcome-file>
    </welcome-file-list>

    <!-- Load spring continer -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring/applicationContext-*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- post encoding -->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- springmvc -->
    <servlet>
        <servlet-name>store-manager</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>store-manager</servlet-name>
        <!-- intercept all requests, include static resources-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

![](images/Screen Shot 2018-04-22 at 2.42.33 PM.png)tt

#### Resource

- item/list
  - [Parameters]: http://localhost:8080/item/list?page=1&amp;amp;amp;amp;amp;rows=30

  - Response -EasyUI datagrid component requires Json format : ***{total: 2, rows:[{"id":1, "name": "zhangsan"},{"id":2, "name": "lisi"}]}***

#### MyBatis PageHelper

MyBatis SqlSessionFactory

​                    |

​            Sql Session

​                    |

​              Exectuor

​                    |      --> MyBatis plugin implements Interceptor interface. Using *LIMIT* in sql statement.

​        MappedStatment

PageHelper.startPage(PAGE_NUM, PAGE_SIZE)

#### Image upload

- Tomcat cluster
- Image server - user upload images, http request
  - Need a http server - e.g. apache, **nginx**
  - Use ftp service to upload image - linux bundled ftp server: **vsftpd**


#### Install Nginx

- PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。注：pcre-devel是使用pcre开发的一个二次开发库。nginx也需要此库。
- zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。
- OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库。

```
ssh hua@10.0.0.100 // start virtualbox node1
apt-cache search nginx
sudo apt-get install nginx
sudo apt-get install libpcre3-dev
sudo apt-get install zlib1g-dev
sudo apt-get install openssl

find / -name nginx // dpkg -L nginx
```

```
/usr/sbin/nginx -s quit
/usr/sbin/nginx -g 'daemon on; master_process on;
/etc/init/nginx.conf 

 apt-get remove --purge 
```

```shell
hua@node1:/usr/sbin$ ps aux | grep nginx
hua       3820  0.0  0.0  21292   936 pts/1    S+   21:50   0:00 grep --color=auto nginx
hua@node1:/usr/sbin$ sudo ./nginx
hua@node1:/usr/sbin$ ps aux | grep nginx
root      3829  0.0  0.1 124976  1428 ?        Ss   21:50   0:00 nginx: master process ./nginx
www-data  3830  0.0  0.3 125336  3192 ?        S    21:50   0:00 nginx: worker process
hua       3833  0.0  0.0  21292   940 pts/1    S+   21:50   0:00 grep --color=auto nginx
hua@node1:/usr/sbin$ sudo ./nginx -s stop
hua@node1:/usr/sbin$ ps aux | grep nginx
hua       3841  0.0  0.1  21292  1012 pts/1    S+   21:50   0:00 grep --color=auto nginx
hua@node1:/usr/sbin$ sudo ./nginx -s reload
```

#### Install ftp

```shell
sudo apt-get install vsftpd
sudo nano /etc/vsftpd.conf

# Uncomment this to enable any form of FTP write command.
write_enable=YES

# You may restrict local users to their home directories.  See the FAQ for
# the possible risks in this before using chroot_local_user or
# chroot_list_enable below.
chroot_local_user=YES
```

open firewall

```shell
hua@node1:/etc$ sudo ufw enable
hua@node1:/etc$ sudo ufw allow 21 //ftp
hua@node1:/etc$ sudo ufw allow 20
hua@node1:/etc$ sudo ufw allow 22 // ssh
hua@node1:/etc$ sudo ufw allow 80 // This has to be allowed once firewall enabled
```

add ftp user

```
hua@node1:/etc$ sudo useradd ftpadmin
hua@node1:/etc$ sudo passwd ftpadmin
Enter new UNIX password: //ftpadmin
Retype new UNIX password: //ftpadmin
passwd: password updated successfully
// setup root directory
hua@node1:/home$ sudo mkdir /home/ftpadmin
hua@node1:/home$ sudo usermod --shell /bin/bash --home /home/ftpadmin ftpadmin
hua@node1:/home$ sudo chown -R ftpadmin:ftpadmin /home/ftpadmin
```

install fileZilla - ftp client, setting -> change ftp to **active**

[Ref]: https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-for-a-user-s-directory-on-ubuntu-16-04

install selinux-utils

```shell
sudo apt install selinux-utils
```

change /etc/vsftpd.config

```shell
write_enable=YES
```

subdirectory configuration — **REAL SHIT!** **TWO NIGHTS OF WORK**

```shell
hua@node1:sudo mkdir /home/ftpadmin/www/images
hua@node1: chmod 755 /home/ftpadmin/www/images
hua@node1: chmod 644 /home/ftpadmin/www/images/*.*
hua@node1:/var/www/html$ sudo vim /etc/nginx/sites-enabled/default
// Add
        # images
        location /images {
                alias /home/ftpadmin/www/images/;
        }
hua@node1:sudo nginx -s stop
hua@node1:sudo nginx
hua@node1:/home/ftpadmin/www$ sudo chmod 777 images
```

#### Note:

1. 使用alias时目录名后面一定要加“/”
2. 一般情况下，在location /中配置root，**在location /other中配置alias**


#### MultipartFile

**springmvc.xml**

```shell
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="defaultEncoding" value="UTF-8"></property>
        <!-- 5MB 5 * 1024 * 1024-->
        <property name="maxUploadSize" value="5242880"></property>
    </bean>
```

**Note:** 

```java
 @RequestMapping("image/upload")
 @ResponseBody // response header - content-type: text/plain
 public String upload(MultipartFile multipartFile){
   return JsonUtils.objectToJson(imageService.upload(multipartFile));
 }


 @RequestMapping("image/upload")
 @ResponseBody // response header - content-type: application/json
 public Map upload(MultipartFile multipartFile){
   return imageService.upload(multipartFile);
 }
```

#### Add item

```javascript
 //$("#itemAddForm").serialize()将表单序列号为key-value形式的字符串
 $.post("/item/add", $("#itemAddForm").serialize(), function (data) 
        if (data.status == 200) {
            $.messager.alert('提示', '新增商品成功!');
        }
 });
```

### Optimized Infrastructure

**Pros:**

- 前台系统和服务层分开，降低耦合度
- 开发团队可以分开，提高开发效率
- 系统可以分开灵活的进行部署

**Cons:**

- 服务之间使用接口通信，开发工作量提高
- 接口通信带来延迟

### Portal Layer

#### Technology Stack

- jstl, jQuery
- Spring
- SpringMVC
- httpClient

### Rest Layer

#### Technology Stack

- Mybatis
- Spring
- Springmvc

#### Cross Domain Origin

- Domain names are different
- Domain names are the same, ports are different

##### JsonP

跨域解决方案。js无法跨域请求，但是可以js跨域请求js脚本。漏洞？我们可以把数据封装成js语句，做一个方法调用。跨域请求的js可以得到此脚本，得到后会立即执行。可以把数据作为参数传递到方法中，从而获取数据解决跨域问题。

### Content Category Mgmt

#### **In order to return auto increased id, have to tweak corresponding mapper.xml method**

```Xml
<insert id="insert" parameterType="com.hua.store.pojo.ContentCategory">
    <selectKey keyProperty="id" resultType="long" order = "AFTER">
      SELECT LAST_INSERT_ID()
    </selectKey>
```

#### store-portal doPost 406

-  检查jackson包是否存在
- 检查后缀是否为html，如果是，修改xml，添加servlet-mapping为action或者do

### Redis Server - 10.0.0.77

#### Install

```shell
sudo apt update
sudo apt full-upgrade
sudo apt install build-essential tcl
sudo apt-get install redis-server
sudo systemctl restart redis-server.service // no need
sudo systemctl stop redis-server.service
sudo systemctl enable redis-server.service
sudo systemctl disable redis-server.service

sudo nano /etc/redis/redis.conf
redis-cli 
```

#### Data Type

- Sting
- Hash
- List
- Set
- SortedSet

#### Cluster Setup

Redis-cluster把所有的物理节点映射到【0-16383】slot上，cluster负责维护 node<—>slot<—>value

*redis集群中内置了16384个哈希槽，当需要在redis集群中放置一个key-value时，redis先对key用crc16算法算出一个结果，然后把结果对16384求余数，这样每个key都会对应一个编号0-16383之间的哈希槽，redis会根据节点数量大致均等的将哈希槽映射到不同节点。*

*领着投票过程是急群众所有master参与，如果半数以上master节点与master节点通信超过（cluster-node-timeout），认为当前master节点挂掉 *

*如果集群任意master挂掉，且当前master没有slave，集群进入fail状态，也可以理解成集群的slot映射【0-16383】不完成时进入fail状态。 redis-3.0.0.rc1加入cluster-require-full-coverage参数，打开集群兼容部分失败。*

*如果集群超过半数以上master挂掉，无论是否有slave集群进入fail状态*。当集群不可用时，所有对集群的操作都不可用，收到((error)CLUSTERDOWN The cluster is down)错误。

#### Cluster Struture

The cluster has 3 nodes, each node has 1 master node and 1 slave node which equals to 6 total. Ideally we need 6 servers.

搭建一个为分布式集群, 使用6个实例来模拟。

```shell
sudo apt install ruby
sudo gem install redis -v 3.0.6 // depends on redis version
```

###### Create Cluster

```shell
hua@node1:~$ cd /usr/local
hua@node1:/usr/local$ sudo mkdir redis-cluster
```

6 redis instance, port number [7001 - 7006]

- change port
- enable cluster-enabled yes
- change pid
- change filename from dump.rdb to redis*.rdb
- change rdb dir to data
- change log file location
- change cluster file location
- change bind to 10.0.0.77
- create redis-trib.rb http://download.redis.io/redis-stable/src/redis-trib.rb

```shell
hua@node1:/usr/local/redis-cluster$ sudo mkdir redis01
hua@node1:/usr/local/redis-cluster$ sudo mkdir redis02
hua@node1:/usr/local/redis-cluster$ sudo mkdir redis03
hua@node1:/usr/local/redis-cluster$ sudo mkdir redis04
hua@node1:/usr/local/redis-cluster$ sudo mkdir redis05
hua@node1:/usr/local/redis-cluster$ sudo mkdir redis06
hua@node1:/usr/local/redis-cluster$ sudo cp /etc/redis/redis.conf /usr/local/redis-cluster/redis01/redis1.conf
hua@node1:/usr/local/redis-cluster$ sudo cp /etc/redis/redis.conf /usr/local/redis-cluster/redis02/redis2.conf
hua@node1:/usr/local/redis-cluster$ sudo cp /etc/redis/redis.conf /usr/local/redis-cluster/redis03/redis3.conf
hua@node1:/usr/local/redis-cluster$ sudo cp /etc/redis/redis.conf /usr/local/redis-cluster/redis04/redis4.conf
hua@node1:/usr/local/redis-cluster$ sudo cp /etc/redis/redis.conf /usr/local/redis-cluster/redis05/redis5.conf
hua@node1:/usr/local/redis-cluster$ sudo cp /etc/redis/redis.conf /usr/local/redis-cluster/redis06/redis6.conf

hua@node1:/usr/local/redis-cluster$ ls
redis01  redis02  redis03  redis04  redis05  redis06
hua@node1:/usr/local/redis-cluster$ sudo mkdir logs
hua@node1:/usr/local/redis-cluster$ sudo mkdir pids
hua@node1:/usr/local/redis-cluster$ sudo mkdir data
hua@node1:/usr/local/redis-cluster$ sudo chmod +rw data
hua@node1:/usr/local/redis-cluster$ sudo chmod 777 logs
hua@node1:/usr/local/redis-cluster$ sudo chmod +rw pids
hua@node1:/usr/local/redis-cluster$ sudo chmod +rx redis01/redis1.conf 
hua@node1:/usr/local/redis-cluster$ sudo chmod +rx redis02/redis2.conf 
hua@node1:/usr/local/redis-cluster$ sudo chmod +rx redis03/redis3.conf 
hua@node1:/usr/local/redis-cluster$ sudo chmod +rx redis04/redis4.conf 
hua@node1:/usr/local/redis-cluster$ sudo chmod +rx redis05/redis5.conf 
hua@node1:/usr/local/redis-cluster$ sudo chmod +rx redis06/redis6.conf 
hua@node1:/usr/local/redis-cluster$ sudo chmod +x redis-trib.rb 
```

安装完成后，可以使用dpkg命令查看各文件所在的路径：

```shell
$ sudo dpkg -L redis-server
```

###### Start-all.sh

```shell
redis-server redis01/redis1.conf
redis-server redis02/redis2.conf
redis-server redis03/redis3.conf
redis-server redis04/redis4.conf
redis-server redis05/redis5.conf
redis-server redis06/redis6.conf
```

```shell
hua@node1:/usr/local/redis-cluster$ sudo chmod +x start-all.sh 
```

执行命令redis-trib.rb，ruby脚本，依赖ruby 

```Ruby
sudo ./redis-trib.rb create --replicas 1 10.0.0.77:7001 10.0.0.77:7002 10.0.0.77:7003 10.0.0.77:7004 10.0.0.77:7005  10.0.0.77:7006
```

redis集群至少三个主节点，每个主节点有一个从节点，一共六个节点

replica指定为1表示每个主节点有一个从节点

```shell
hua@node1:/usr/local/redis-cluster$ sudo ./start-all.sh 
hua@node1:/usr/local/redis-cluster$ sudo ./redis-trib.rb create --replicas 1 10.0.0.77:7001 10.0.0.77:7002 10.0.0.77:7003 10.0.0.77:7004 10.0.0.77:7005  10.0.0.77:7006
./redis-trib.rb:1573: warning: key "threshold" is duplicated and overwritten on line 1573
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
10.0.0.77:7001
10.0.0.77:7002
10.0.0.77:7003
Adding replica 10.0.0.77:7004 to 10.0.0.77:7001
Adding replica 10.0.0.77:7005 to 10.0.0.77:7002
Adding replica 10.0.0.77:7006 to 10.0.0.77:7003
M: 3215aa852927b74da7790634ad2ae848cef920d6 10.0.0.77:7001
   slots:0-5460 (5461 slots) master
M: e09ec3d369a5cb082d1aea9132a378ff2c4b70d0 10.0.0.77:7002
   slots:5461-10922 (5462 slots) master
M: f0d9a45edcbef011527a1cc1b5efb9afd1ba4c64 10.0.0.77:7003
   slots:10923-16383 (5461 slots) master
S: 5b87bb63781591fbae1159b8513205f9581aad88 10.0.0.77:7004
   replicates 3215aa852927b74da7790634ad2ae848cef920d6
S: 0eafc4b443ac03b743e26ec13717735d9d55ce18 10.0.0.77:7005
   replicates e09ec3d369a5cb082d1aea9132a378ff2c4b70d0
S: b658a243a08c6e4840ea0c3632750ddb28a213c1 10.0.0.77:7006
   replicates f0d9a45edcbef011527a1cc1b5efb9afd1ba4c64
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.....
>>> Performing Cluster Check (using node 10.0.0.77:7001)
M: 3215aa852927b74da7790634ad2ae848cef920d6 10.0.0.77:7001
   slots:0-5460 (5461 slots) master
M: e09ec3d369a5cb082d1aea9132a378ff2c4b70d0 10.0.0.77:7002
   slots:5461-10922 (5462 slots) master
M: f0d9a45edcbef011527a1cc1b5efb9afd1ba4c64 10.0.0.77:7003
   slots:10923-16383 (5461 slots) master
M: 5b87bb63781591fbae1159b8513205f9581aad88 10.0.0.77:7004
   slots: (0 slots) master
   replicates 3215aa852927b74da7790634ad2ae848cef920d6
M: 0eafc4b443ac03b743e26ec13717735d9d55ce18 10.0.0.77:7005
   slots: (0 slots) master
   replicates e09ec3d369a5cb082d1aea9132a378ff2c4b70d0
M: b658a243a08c6e4840ea0c3632750ddb28a213c1 10.0.0.77:7006
   slots: (0 slots) master
   replicates f0d9a45edcbef011527a1cc1b5efb9afd1ba4c64
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

##### Test Cluster 

###### with -c

```shell
hua@node1:~$ redis-cli -h 10.0.0.77 -p 7003 -c //cluster
hua@node1:~$ redis-cli -h 10.0.0.77 -p 7006 -c //cluster
10.0.0.77:7006> set name hualiang
-> Redirected to slot [5798] located at 10.0.0.77:7002
OK
```

###### without -c

```shell
hua@node1:~$ redis-cli -h 10.0.0.77 -p 7003 -c //cluster
hua@node1:~$ redis-cli -h 10.0.0.77 -p 7006 -c //cluster
10.0.0.77:7006> set name hualiang
(error) MOVED 5798 10.0.0.77:7002
```

#### 缓存同步

当后台管理系统修改内容之后，需要通知redis修改其cache内容（内容对应的key删除），否则用户永远查不到最新数据。

**解决方案**

Store-api中定义服务，当后台管理系统修改内容时，调用此服务，清空缓存.

### Solr

#### 方案

##### 第一种方案(不可行)

**Install Java8**

```shell
apt-get update && apt-get upgrade
apt-get install default-jdk // java8
```

**Install tomcat**

```shell
hua@node1:sudo apt install curl
hua@node1:sudo mkdir /usr/local/solr

hua@node1:/usr/local/solr$ sudo wget http://www-eu.apache.org/dist/tomcat/tomcat-8/v8.5.31/bin/apache-tomcat-8.5.31.tar.gz
```

**Install Solr**

```Shell
hua@node1:/usr/local/solr$ sudo wget http://archive.apache.org/dist/lucene/solr/6.6.2/solr-6.6.2.tgz
hua@node1:/usr/local/solr$ sudo tar -zxvf solr-6.6.2.tgz 

hua@node1:/usr/local/solr/solr-6.6.2/server/solr-webapp$ cp webapp/ /usr/local/solr/tomcat-8.5.31/webapps/solr -r //rename
hua@node1:/usr/local/solr/solr-6.6.2/server/lib$ sudo cp *.jar /usr/local/solr/tomcat-8.5.31/webapps/solr/WEB-INF/lib/ // unnecessary
hua@node1:/usr/local/solr/solr-6.6.2/server/lib/ext$ sudo cp *.jar /usr/local/solr/tomcat-8.5.31/webapps/solr/WEB-INF/lib/
hua@node1:/usr/local/solr/solr-6.6.2/dist$ sudo cp solr-dataimporthandler-6.6.2.jar solr-dataimporthandler-extras-6.6.2.jar /usr/local/solr/tomcat-8.5.31/webapps/solr/WEB-INF/lib/
hua@node1:/usr/local/solr/solr-6.6.2/server$ sudo cp -R solr /usr/local/solr/solrhome
```

**Configuration**

```shell
hua@node1:/usr/local/solr/tomcat-8.5.31/webapps/solr/WEB-INF$ sudo vim web.xml 
<env-entry>
       <env-entry-name>solr/home</env-entry-name>
       <env-entry-value>/usr/local/solr/solrhome</env-entry-value>
       <env-entry-type>java.lang.String</env-entry-type>
 </env-entry>
 
hua@node1:/usr/local/solr/tomcat-8.5.31/webapps/solr/WEB-INF$ sudo mkdir classes

// This command is not necessarily needed, causing solr.dir not found exception
hua@node1:/usr/local/solr/solr-6.6.2/server/resources$ sudo cp *.properties /usr/local/solr/tomcat-8.5.31/webapps/solr/WEB-INF/classes/log4j.properties

hua@node1:/usr/local/solr$ sudo chmod -R 777 tomcat-8.5.31/
hua@node1:/usr/local/solr/solr-7.3.0/server/solr$ sudo cp * /usr/local/solr/solrhome/
hua@node1:/usr/local/solr/solrhome$ mkdir logs
hua@node1:/usr/local/solr/solrhome$ chmod 777 logs

hua@node1:/usr/local/solr/tomcat-8.5.31/bin$ ./startup.sh
Using CATALINA_BASE:   /usr/local/solr/tomcat-8.5.31
Using CATALINA_HOME:   /usr/local/solr/tomcat-8.5.31
Using CATALINA_TMPDIR: /usr/local/solr/tomcat-8.5.31/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /usr/local/solr/tomcat-8.5.31/bin/bootstrap.jar:/usr/local/solr/tomcat-8.5.31/bin/tomcat-juli.jar
Tomcat started.

```

[url]: http://10.0.0.97:8080/solr/index.html#/

##### 第二种方案(不可行)

1. 下载solr

2. 修改web.xml中的SOLR_HOME

3. bin文件夹下启动 ./solr start

4. [url]: http://10.0.0.97:8983/

```shell
hua@node1:/usr/local/solr/solr-7.3.0/server$ sudo mkdir logs
hua@node1:/usr/local/solr/solr-7.3.0/server$ sudo chmod 777 logs
```

##### 第三种方案(可行)

1. 下载solr-4.9.1，解压
2. 创建solrhome
3. 下载tomcat

```Shell
hua@node1:/usr/local/solr$ sudo wget http://archive.apache.org/dist/lucene/solr/4.9.1/solr-4.9.1.tgz
hua@node1:/usr/local/solr$ sudo tar zxf solr-4.9.1.tgz 

hua@node1:/usr/local/solr$ sudo wget http://www-us.apache.org/dist/tomcat/tomcat-7/v7.0.86/bin/apache-tomcat-7.0.86.tar.gz
hua@node1:/usr/local/solr$ sudo tar -zxf apache-tomcat-7.0.86.tar.gz
hua@node1:/usr/local/solr$ sudo mv apache-tomcat-7.0.86 tomcat-7.0.86

hua@node1:/usr/local/solr/solr-4.9.1/example/webapps$ sudo cp solr.war /usr/local/solr/tomcat-7.0.86/webapps/solr.war

hua@node1:/usr/local/solr/tomcat-7.0.86/bin$ sudo ./startup.sh 
hua@node1:/usr/local/solr/tomcat-7.0.86/bin$  tail -f ../logs/catalina.out 

hua@node1:/usr/local/solr/tomcat-7.0.86/bin$ sudo ./shutdown.sh //必须先关闭， 否则solr文件夹也被删除
hua@node1:/usr/local/solr/tomcat-7.0.86/webapps$ sudo rm -rf solr.war 
hua@node1:/usr/local/solr/solr-4.9.1/example/lib/ext$ sudo cp * /usr/local/solr/tomcat-7.0.86/webapps/solr/WEB-INF/lib/
hua@node1:/usr/local/solr/solr-4.9.1/example/solr$ sudo cp -r * /usr/local/solr/solrhome/
hua@node1:/usr/local/solr/tomcat-7.0.86/webapps/solr/WEB-INF$ sudo vim web.xml

<env-entry>
    <env-entry-name>solr/home</env-entry-name>
    <env-entry-value>/usr/local/solr/solrhome</env-entry-value>
    <env-entry-type>java.lang.String</env-entry-type>
</env-entry>

hua@node1:/usr/local/solr/solr-4.9.1/example/lib$ cd /usr/local/solr/tomcat-7.0.86/webapps/solr/WEB-INF/lib/ // Needed? 
```

#### Analyser

##### IK-Analyzer 中文分词器

- 将扩展词典，停用词典，配置文件复制到solr工程的classpath。字符集必须是utf-8，不能使用windows记事本编辑。
- 配置field type，需要在solrhome/collection1/conf/schema.xml中配置。

```shell
Huas-MacBook-Pro-2:IK Analyzer 2012 hliang$ scp IKAnalyzer2012FF_u1.jar hua@10.0.0.97:/usr/local/solr/tomcat-7.0.86/webapps/solr/WEB-INF/lib
Huas-MacBook-Pro-2:IK Analyzer 2012 hliang$ scp IKAnalyzer.cfg.xml mydict.dic ext_stopword.dic hua@10.0.0.97:/usr/local/solr/tomcat-7.0.86/webapps/solr/WEB-INF/classes


hua@node1:/usr/local/solr/solrhome/collection1/conf$ sudo vim schema.xml // GO to the end - G, add following

<fieldType name="text_ik" class="solr.TextField">
    <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
</filedType>
```

##### 业务字段配置

- 搜索时是否要在此字段进行搜索，比如： 项目名称，项目卖点，项目描述
- 后续业务是否需要用到此字段，例如：商品id

##### 需要用到的字段

- 商品 id（solr业务字段 id —> 商品id）
- 商品title
- 商品价格
- 商品卖点
- 商品图片
- 商品分类
- 商品描述

```xml
hua@node1:/usr/local/solr/solrhome/collection1/conf$ sudo vim schema.xml //末尾添加
    <fieldType name="text_ik" class="solr.TextField">
       <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
    </fieldType>
    <field name="item_title" type="text_ik" indexed="true" stored="true"/>
    <field name="item_sell_point" type="text_ik" indexed="true" stored="true"/>
    <field name="item_price" type="long" indexed="true" stored="true"/>
    <field name="item_image" type="string" indexed="true" stored="true"/>
    <field name="item_category_name" type="string" indexed="true" stored="true"/>
    <field name="item_desc" type="text_ik" indexed="true" stored="false"/>
    <field name ="item_keywords" type="text_ik" indexed="true" stored="false" multiValued="true"/>
    <copyField source="item_title" dest="item_keywords"/>
    <copyField source="item_sell_point" dest="item_keywords"/>
    <copyField source="item_category_name" dest="item_keywords"/>
    <copyField source="item_desc" dest="item_keywords"/>
```

##### 维护索引

- 添加：添加一个json格式文件即可。

- 修改：在solr中没有update，只要添加一个新的文档，要求文档id和被修改的id。原理是先删除后添加。

- 删除：

  - 根据id删除

    ```
    <delete>
    	<id>test001</id>
    </delete>
    </commit>
    ```

  - 根据query

    ```
    <delete>
    	<query>*:*</query>
    </delete>
    </commit>
    ```

#### Solr Client

需要添加pom.xml

```Xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpmime</artifactId>
    <version>4.2.3</version>
</dependency>
```

为了灵活进行分布式部署，需要创建一搜索的服务工程发布搜索服务 - store-search.

- war - sping, springmvc
- 无transcation，无需applicationContext-transaction.xml
- 无redis连接

#### Tables

- tb_item
- tb_item_content
- tb_desc

```sql
SELECT 
    i.id,
    i.title,
    i.sell_point,
    i.price,
    i.image,
    ic.name AS category_name,
    id.item_desc
FROM
    tb_item i
        LEFT JOIN
    tb_item_cat ic ON i.cid = ic.id
        LEFT JOIN
    tb_item_desc id ON i.id = id.item_id
```

#### 搜索服务

http协议，对外提供get形式的服务。

请求url：/search?q={查询条件} was &page={page}&row={row}

解决get乱码：new String(query.getBytes("iso8859-1"), "utf-8")

[查询]: http://localhost:8083/search/query?q=%E6%89%8B%E6%9C%BA&amp;amp;amp;amp;amp;rows=30&amp;amp;amp;amp;amp;page=5

#### 缓存逻辑

Redis的hash类型key是不能设置过期时间。如果还需要对key进行分类可以使用折中方式。

store:hua:test, 层级结构，可以对应表中字段。

商品key的定义：

REDIS_ITEM_KEY：商品 id:base=json

REDIS_ITEM_KEY：商品 id:desc=json

REDIS_ITEM_KEY：商品 id:param=json

### Session共享

#### 第一种方案

配置tomcat的session共享。配置tomcat集群。Tomcat配置好集群后，会不停向集群中的其他tomcat广播自己的session信息。其他tomcat做session同步。保证session的一致性。

**优点**：不需额外开发（修改代码），只需搭建tomcat集群即可。

**缺点**：

- tomcat是全局session复制，集群内每个tomcat的session完全同步，在大规模应用的时候如果用户过多，集群内tomcat数量过多，session的全局复制会导致集群性能下降，因此，tomcat的数量不能太多，最好5个以下。
- 无法解决分布式session共享问题。例如：支付宝和淘宝。

#### 第二种方案

实现单点登录系统（sso）

实现单点登录系统，提供服务接口。把session数据存放在redis中。

redis可以设置key的生存时间，访问速度快效率高。

**优点**：redis存取速度快，不会出现多个节点session复制问题。效率高。

**缺点**：需要程序员开发。

#### 单点登录系统

单点登录系统是一个独立工程。需要操作redis，连接数据库。

##### **需要使用的技术**

- mybatis - 只需要依赖mapper
- spring
- springmvc
- redis

**note**: form-post: x-www-form-urlencoded

## 链接

[后台]: http://localhost:8080/
[门户]: http://localhost:8082/
[API ]: http://localhost:8081/api/
[索引]: http://localhost:8083/search/manager/importall
[索引]: http://10.0.0.97:8080/solr/#/collection1/query	"df:item_keywords"
[sso]: http://localhost:8084//user/check/zhangsan/1?callback=callbackfunc



## Reference

[mvc:resources]: https://docs.spring.io/spring/docs/3.0.x/spring-framework-reference/html/mvc.html#mvc-static-resources
[mybatis page helper]: https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/en/HowToUse.md
[easyui的datagrid對應的java對象]: https://hk.saowen.com/a/a2afa859baee4c35d5ee46363513d2630a5bfd7259564334480affa4c6546ee2
[nginx配置静态文件目录404]: https://zhangguodong.me/2017/01/22/nginx%E9%85%8D%E7%BD%AE%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6%E7%9B%AE%E5%BD%95404/
[How To Install and Configure Redis on Ubuntu 16.04]: https://www.rosehosting.com/blog/how-to-install-configure-and-use-redis-on-ubuntu-16-04
[redis数据分布及槽信息]: https://www.jianshu.com/p/b35d778fa529
[How to Install and Configure a Redis Cluster on Ubuntu]: https://www.linode.com/docs/applications/big-data/how-to-install-and-configure-a-redis-cluster-on-ubuntu-1604
[How to Install Java on Ubuntu]: https://thishosting.rocks/install-java-ubuntu/
[How To Install Apache Tomcat 8 on Ubuntu 16.04]: https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-ubuntu-16-04
[solr6.4.1安装部署至tomcat教程]: https://my.oschina.net/zycn/blog/841762
[ik-analyzer]: https://code.google.com/archive/p/ik-analyzer/downloads
[Spring和springmvc父子容器注解扫描问题详解]: https://blog.csdn.net/u011217058/article/details/76076352

## TroubleShoot

[java.lang.ClassNotFoundException: com.fasterxml.jackson.core.JsonProcessingException]: https://blog.csdn.net/RyanDYJ/article/details/76687161
[PageHelper Cannot cast to Interceptor. #48]: https://github.com/pagehelper/Mybatis-PageHelper/issues/48
[Could not chdir to home directory /home/me: No such file or directory]: https://askubuntu.com/questions/401201/could-not-chdir-to-home-directory-home-me-no-such-file-or-directory?utm_medium=organic&amp;amp;amp;amp;amp;utm_source=google_rich_qa&amp;amp;amp;amp;amp;utm_campaign=google_rich_qa
[uploading files from my computer to an ubuntu server]: https://stackoverflow.com/questions/42723895/uploading-files-from-my-computer-to-an-ubuntu-server?utm_medium=organic&amp;amp;amp;amp;amp;utm_source=google_rich_qa&amp;amp;amp;amp;amp;utm_campaign=google_rich_qa
[Nginx 403 error: directory index of [folder\] is forbidden]: https://stackoverflow.com/questions/19285355/nginx-403-error-directory-index-of-folder-is-forbidden?utm_medium=organic&amp;amp;amp;amp;utm_source=google_rich_qaamp;utm_campaign=google_rich_qa

