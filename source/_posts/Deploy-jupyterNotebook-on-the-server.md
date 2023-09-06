---
title: 服务器部署Jupyter（远程登陆）
date: 2023-08-12 19:39:00
tags: Jupyter
description: 服务器部署Jupyter Notebook并配置远程登录
---





# 安装Jupyter

> notebook环境

```bash
pip install jupyter notebook
```



# 配置

> jupyter notebook 配置

```bash
jupyter notebook --generate-config

# Writing default config to: /home/ubuntu/.jupyter/jupyter_notebook_config.py
```



> 创建密码（自动配置）

终端输入jupyter notebook password，

Enter password: xxxx
Verify password: xxxx

成功后显示 `Wrote hashed password to /home/ubuntu/.jupyter/jupyter_server_config.json`



> 修改jupyter notebook的配置文件

vim ~/.jupyter/jupyter_notebook_config.py

在该文件中做如下修改或直接在文件尾端添加：

```python
c.NotebookApp.allow_remote_access = True #允许远程连接
c.NotebookApp.ip='*' # 设置所有ip皆可访问
c.NotebookApp.open_browser = False # 禁止自动打开浏览器
c.NotebookApp.port =8888 #任意指定一个端口
```



# 启动

> 启动Jupyter

终端输入

```bash
jupyter notebook
```



使用**nohup**后台运行

```bash
nohup jupyter notebook >~/jupyter.log 2>&1 &
```



> 远程访问

浏览器输入

```
服务器ip:端口号
```

![image-20230812195930653](./Deploy-jupyterNotebook-on-the-server/image-20230812195930653.png)

输入密码即可访问



