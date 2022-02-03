[TOC]



## nginx常用命令

 

### 常用命令

 查看 Nginx 启动状态 

```shell
ps -ef | grep nginx
```

 检查配置文件nginx.conf的正确性命令 

```shell
/usr/local/webserver/nginx/sbin/nginx -t
```

启动 Nginx

```shell
/usr/local/webserver/nginx/sbin/nginx
```

Nginx 其他命令

```shell
/usr/local/webserver/nginx/sbin/nginx -s reload            # 重新载入配置文件
/usr/local/webserver/nginx/sbin/nginx -s reopen            # 重启 Nginx
/usr/local/webserver/nginx/sbin/nginx -s stop              # 停止 Nginx
```

### 安装配置教程

[nginx官方网站](http://nginx.org/en/)

[菜鸟ngnix安装教程](https://www.runoob.com/linux/nginx-install-setup.html)

[nginx github站点](https://github.com/nginx/nginx)

[nginx中文文档](https://www.nginx.cn/doc/)

 

### nginx可视化配置项目

[nginxconfig.io](https://github.com/digitalocean/nginxconfig.io)

[nginxWebUI](https://gitee.com/cym1102/nginxWebUI)
