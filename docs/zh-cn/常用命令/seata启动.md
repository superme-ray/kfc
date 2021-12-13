## seata 配置守护进程





> 1、首先在/data/cmp/seata/seata目录下新增启动脚本startup.sh ；



编辑添加 

```shell
#!/bin/bash
~ sh /data/cmp/seata/seata/bin/seata-server.sh -p 8091 -h 127.0.0.1
```



> 2、对该脚本授权：

```shell
~ chmod 777 /data/cmp/seata/seata/startup.sh
```



> 3、然后在/usr/lib/systemd/system目录下新建seata.service ；命令 ：vi seata.service

编辑添加内容：

```shell
[Unit]
Description=seata-server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=simple
ExecStart=/data/cmp/seata/seata/startup.sh
Restart=always
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```



> 4、对该脚本授权：

```shell
~ chmod 777 /usr/lib/systemd/system/seata.service
```



> 5、启用该服务

```shell
~ systemctl enable seata.service
~ systemctl daemon-reload
```



> 6、守护进程启动seata和查看状态，命令如下：

```shell
~ systemctl start seata.service
~ ps -ef|grep seata
```



> 7、关闭客户端后依然能连接到seata服务，证明配置成功。 



> 8、systemctl service自定义注册服务的一些参数：systemctl管理的服务脚本存放在/usr/lib/systemd/,有user与systemd区分需要开机自启动存放在system目录下：/usr/lib/systemd/system 并且服务的脚本一般以.service结尾




> 9、命令
```shell
~ systemctl start seata.service
~ systemctl stop seata.service
~ systemctl reload seata.service
~ systemctl status seata.service
```



