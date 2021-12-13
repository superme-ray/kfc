## druid 数据库加密方法



#### 利用配置完成加密

1. 利用druid的jar包生成对应的publicKey，privateKey，password

   ```shell
   ~ java -cp D:\environment\apache-maven-3.6.3\repository\com\alibaba\druid\1.2.4\druid-1.2.4.jar com.alibaba.druid.filter.config.ConfigTools 123456
   
   ```

   执行命令后生成对应的值：

   ```json
   privateKey:MIIBVgIBADANBgkqhkiG9w0BAQEFAASCAUAwggE8AgEAAkEAiNTHXZbTodDI86PXYVH8vh4JnYoYbNTUBWjRFZrzKgnuQtma/2XNNL16a4K0dtJrRmur/HUDZGtJsBJYJaIo3wIDAQABAkAMwbmskgk9BtgVTusfmaM0nlxLIbrROq5hqroDh6SwAIYdO2sHyp/lSrHzRPhVqinrcyLSO3zmH1wiA
   gZoYfKxAiEAvdqlMAr3JqJ5zlVylnJ6JE1ihQRuLQ13z1MKGeiCVlUCIQC4gPHJwyyMQzgOYbFxc8YnD/OwVrRVZG57OlVnHVmuYwIhAIXnQ2DKKx0NtVlo/OPNpAYcqmLlCAwwlpMcn2A8lEjtAiEArsefxNT6J2kp+h27nVDiPnDTFZIdRONd8ahB7OuV4CcCIQCjm3CKZhuxeCUJOq8+HQQ8XxWf/0UeMSIBD
   b9rAA0h8Q==
   publicKey:MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAIjUx12W06HQyPOj12FR/L4eCZ2KGGzU1AVo0RWa8yoJ7kLZmv9lzTS9emuCtHbSa0Zrq/x1A2RrSbASWCWiKN8CAwEAAQ==
   password:B0HU/gB6puPrjMg4EHs9Fw8GdW7rr4i9XCsbmTuQYtxw3SslSkCc1EjTt5fP/EYuuwltf3ls4oFbCbh8gO9+Bg==
   
   ```

2. 将生成的publicKey， password配置到配置文件中

   ```yaml
   spring:
     datasource:
       type: com.alibaba.druid.pool.DruidDataSource
       driver-class-name: com.mysql.cj.jdbc.Driver
       druid:
         username: test_edu
         password: pU6kFhGmzI1DVeFtvIVBberT86KsIUE/iPQeN7wsqHyYNAnvQ/2GZBbOCvdm7SJpaC3klmqtCdvfDtk1FLocQA==
         url: jdbc:mysql://rm-2vctv30j64a5023n7qo.mysql.cn-chengdu.rds.aliyuncs.com:3306/t-education?characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowMultiQueries=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=Asia/Shanghai
         filter:
           config:
             enabled: true
         connection-properties: config.decrypt=true;config.decrypt.key=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAMUKJrfaFd7QgLYKdU6/auBBAHn7DgRZKRdMo6yRqcaAWLjudPH4OP58A3EgMc/GFlvHjVK3pF/Qz/FTVKBtPksCAwEAAQ==
         filters: stat,wall
   ```

3. 进行程序连接



#### 自定义key完成加密

> 自定义加密需要加一个重写密码解密的类

1. 新建MyDruidPwd.java

```java
package com.education.config;

import com.alibaba.druid.util.DruidPasswordCallback;
import lombok.extern.slf4j.Slf4j;

import java.util.Properties;


/**
 * @author ray
 * @version 1.0
 * @date 2021/4/15 13:41
 */
@Slf4j
public class MyDruidPwd extends DruidPasswordCallback {

    public static final String key = "-education";

    @Override
    public void setProperties(Properties properties) {
        super.setProperties(properties);
        char[] chars = null;
        try {
            String ciphertext = properties.getProperty("pwd");
            //安装之前密码加密的方式进行解密
            String pwd = encryptPwd(ciphertext);
            chars = pwd.toCharArray();
        } catch (Exception e) {
            e.printStackTrace();
            log.info("解密失败,{}", e.getMessage());
        }
        super.setPassword(chars);
    }

    /**
     * 自定义解密
     *
     * @param pwd
     * @return
     */
    private String encryptPwd(String pwd) {
        //自定义加密，可用md5加密
        //TODO
        return pwd.split(key)[0];
    }

    /**
     * 自定义加密
     * @param ciphertext
     * @return
     */
    private static String decryptPwd(String ciphertext) {
        //自定义解密，按照加密方式进行解密
        //TODO
        return ciphertext + key;
    }

    public static void main(String[] args) {
        System.out.println(decryptPwd("pU6kFhGmzI1DVeFtvIVBberT86KsIUE/iPQeN7wsqHyYNAnvQ/2GZBbOCvdm7SJpaC3klmqtCdvfDtk1FLocQA=="));
    }

}

```


2. yml配置

   ```yaml
   spring:
     datasource:
       type: com.alibaba.druid.pool.DruidDataSource
       driver-class-name: com.mysql.cj.jdbc.Driver
       druid:
         username: test_edu
         password:
         url: jdbc:mysql://rm-2vctv30j64a5023n7qo.mysql.cn-chengdu.rds.aliyuncs.com:3306/t-education?characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowMultiQueries=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=Asia/Shanghai
         connection-properties: pwd=test_edu888Cyjy-education
         filters: stat,wall
         password-callback-class-name: com.education.config.MyDruidPwd
   ```

3. 解密过程

   在自定义类的重写方法中按照自定义加密规则进行解密再赋值给druid