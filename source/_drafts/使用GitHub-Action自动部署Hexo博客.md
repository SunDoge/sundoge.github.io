---
title: 使用GitHub Action自动部署Hexo博客
tags:
---

基本流程参照[https://github.com/marketplace/actions/hexo-action](https://github.com/marketplace/actions/hexo-action)

## Step 1. 生成ssh key pair

首先，我们需要生成ssh key pair。最好是专门为这个仓库生成一个pair，因为后面需要上传私钥，如果是本机常用的ssh key pair，会存在一定风险。

```bash
ssh-keygen -t rsa -C "username@example.com"
```

邮箱可以修改为自己的邮箱，询问保存key的文件路径时，随便写一个，避免覆盖原有的key。

```bash
 ± ssh-keygen -t rsa -C "username@example.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/sundoge/.ssh/id_rsa): hexo
```

以输入`hexo`为例，在当前目录下得到`hexo`，`hexo.pub`两个文件，前者是私钥，后者是公钥。



![image.png](https://i.loli.net/2020/07/31/C9HbrLjyUO1X3EQ.png)