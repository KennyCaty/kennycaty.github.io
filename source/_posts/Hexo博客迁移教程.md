---
title: Hexo博客迁移教程
date: 2023-07-20 19:43:29
tags: 个人博客
---

# Hexo博客迁移

> 换电脑后如何将原来的Hexo博客迁移？
>
> 解决方案：将原本地文件存放至GitHub博客项目的新分支，需要更新本地文件时pull或者push
>
> 参考博客：[Hexo_博客迁移问题（换电脑） - 向南风 ](https://www.cnblogs.com/itvdo/p/11323937.html)



## 1.本地文件存放至GitHub

1.1 克隆自己的XXX.github.io项目到本地

```bash
git clone https://github.com/yourname/xxx.github.io.git
```

1.2 删除文件夹里**除了.git的**其他所有文件（这个文件是隐藏文件)

1.3 将原Hexo本地项目文件夹中除了**node_modules**的其他文件全部复制到这个文件夹中

​		node_modules可以后面重新通过npm install部署，换电脑的时候可以直接删掉

1.4 确认一下里面是否有一个文件.gitignore，如果没有就输入`touch .gitignore` 直接创建一个，然后粘贴如下内容，保存。

```bash
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```



1.5 **在原博客GitHub项目中创建一个新分支**，我这里取名为hexo

![image-20230720185828386](E:\kennycaty.github.io\source\_posts\Hexo博客迁移教程\image-20230720185828386.png)

本地也创建一个分支，并切换到这个分支上

```bash
git checkout -b hexo
```



1.6 提交，推送到github指定新分支

```bash
git add --all
git commit -m "新建资源文件存放分支"
git push --set-upstream origin hexo
```





## 2.新电脑拉取本地项目文件，使用Hexo重新生成，部署

2.1 重新安装Hexo

使用国内源镜像**cnpm**：

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

下载hexo-cli：

```bash
cnpm install -g hexo-cli
```

hexo -v 验证是否安装成功

![image-20230720190857706](E:\kennycaty.github.io\source\_posts\Hexo博客迁移教程\image-20230720190857706.png)

**2.2 克隆hexo分支（项目源文件）到本地**

注意要选择分支克隆！！

```bash
git clone -b hexo https://github.com/yourname/xxx.github.io.git
```



2.3 安装依赖 `npm install`



然后就可以正常使用 `hexo g` ， `hexo d `，`hexo s`等命令啦 

以后要更新备份本地文件，就可以直接`git push`和` git pull`

