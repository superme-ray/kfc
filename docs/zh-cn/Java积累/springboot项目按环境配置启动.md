## springboot项目按环境配置启动

> 新建的springboot项目只会有一个application.properties配置文件,实际开发时是按照不同的环境配置一个配置文件。

 两中方式可以实现：

#### 第一种，因为springboot本身不支持bootstrap.yml配置，所以按照springboot项目启动加载配置文件的顺序实现
 > 1. 在项目中新增application-local.properties、application-dev.properties2个文件。
 > 2. 分别在对应的文件中写上配置
 > 3. 在application.properties中写上如下配置
 >
    server:
      port: 8005

    spring:
      application:
        name: server-name
      profiles:
        active: prod



#### 第二种，引入 springcloud 的组件依赖，添加bootstrap.yml文件
 > 1. 添加pom依赖
```
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-context</artifactId>
   </dependency>
```


 > 2. 添加bootstrap.yml文件。写上以下代码：
 >
   ```yml
   server:
     port: 8006

   spring:
     application:
       name: server-name
     profiles:
       active: local

   # local
   ---
   ## 配置中心
   spring:
     profiles: local


   # dev
   ---
   # 配置中心
   spring:
     profiles: dev


   # prod
   ---
   # 配置中心
   spring:
     profiles: prod
```
 > 3. 在分别新建2个配置文件application-local.yml、   application-dev.yml
 > 4. 分别写上各自的配置即可
