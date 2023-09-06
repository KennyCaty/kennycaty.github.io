---
title: MySQL-Install
date: 2023-08-07 18:51:06
tags: MySQL, Database, Install
categories: Database
description: 记录自己使用的的MySQL安装文档 2023.8.7更新
---



# 下载

> 当前版本：8.10.0
>
> [MySQL下载地址](https://dev.mysql.com/downloads/mysql/)
>
> 选择解压版（第二个）下载

![image-20230807185436832](./MySQL-Install/image-20230807185436832.png)



# 解压

解压到自己的目录下

![image-20230807190156920](./MySQL-Install/image-20230807190156920.png)



# 配置

## 配置环境变量

系统变量中新建`MYSQL_HOME`

![image-20230807190613079](./MySQL-Install/image-20230807190613079.png)



在 `PATH` 中添加bin目录

![image-20230807190910972](./MySQL-Install/image-20230807190910972.png)

## 打开命令行验证

命令行**以管理员身份运行**

```bash
mysql
```

![image-20230807191324515](./MySQL-Install/image-20230807191324515.png)

输出如下，则配置成功



## 初始化MySQL

在打开的命令行（管理员）中输入：

```bash
mysqld --initialize-insecure
```

成功运行后会在安装目录下生成data文件夹，用于存放MySQL数据

![image-20230807191807294](./MySQL-Install/image-20230807191807294.png)





## 注册MySQL服务

命令行（管理员）：

```bash
mysqld -install
```

![image-20230807192018042](./MySQL-Install/image-20230807192018042.png)

成功后就可以在服务中找到mysql，**默认开机自动启动**

![image-20230807192135486](./MySQL-Install/image-20230807192135486.png)



## 启动改为手动（可选）

为了加快开机速度，可以设置手动启动服务，不过以后使用前**记得手动开启MySQL**

右键我的电脑->管理->服务与应用程序->服务，找到mysql将启动类型改为手动

![image-20230807192216311](./MySQL-Install/image-20230807192216311.png)



## 启动MySQL服务

命令行（管理员）：

```bash
net start mysql //启动服务
```

```bash
net stop mysql //停止服务
```

或者服务设置里面手动点击启动/关闭

![image-20230807192455177](./MySQL-Install/image-20230807192455177.png)

![image-20230807192522745](./MySQL-Install/image-20230807192522745.png)





## 修改MySQL默认账号密码

命令行（管理员）：

```bash
mysqladmin -u root password 123456
```

root：默认管理员权限（自带）  修改密码为自己喜欢的

![image-20230807192744103](./MySQL-Install/image-20230807192744103.png)



# 登录MySQL

命令行输入

```bash
mysql -uroot -p123456
```

![image-20230807193052668](./MySQL-Install/image-20230807193052668.png)

退出时输入 `exit`



> mysql连接指定服务器IP地址, 注意后面的P是大写！

```bash
mysql -u用户名 -p密码 [-h数据库服务器IP -P端口号]
```



# 卸载MySQL

命令行（管理员）

* 停止服务

```bash
net stop mysql
```

* remove mysql

```bash
mysqld -remove mysql
```

* 最后删除MySQL目录以及相关环境变量

卸载完成！
