[TOC]

## redis常用命令

为了安全，服务端redis要设置密码

### Windows下

**redis服务端启动**

在redis安装目录下使用下面命令

```bash
redis-server.exe redis.windows.conf
```

**客户端启动**

在redis安装目录下使用下面命令

```bash
redis-cli.exe -h 127.0.0.1 -p 6379
```

 **设置键值对**

```bash
set myKey abc
```

 **取出键值对**

```bash
get myKey
```

 

### Linux下

**redis服务端启动**

源码安装后， **src** 目录下会出现编译后的 redis 服务程序 redis-server，还有用于测试的客户端程序 redis-cli 

```bash
$ cd src
$ ./redis-server
```

**客户端启动**

```bash
$ cd src
$ ./redis-server ../redis.conf
```

 

**在远程服务上执行命令**

命令格式

```bash
redis-cli -h host -p port -a password
```



```bash
$redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING

PONG
```

 

### 安装使用教程

[菜鸟redis教程](https://www.runoob.com/redis/redis-install.html)

[redis官网](https://redis.io/)

[redis github站点](https://github.com/redis/redis)

