---
title: 解决shallow update not allowed的问题
date: 2020-08-04 14:53:44
tags:
  - Git
---

今天push代码的时候遇到一个问题：

```
Counting objects: 47, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (44/44), done.
Writing objects: 100% (47/47), 573.15 KiB | 0 bytes/s, done.
Total 47 (delta 11), reused 0 (delta 0)
remote: Resolving deltas: 100% (11/11), done.
To github.com:SunDoge/xxxx.git
 ! [remote rejected] master -> master (shallow update not allowed)
error: failed to push some refs to 'git@github.com:SunDoge/xxxx.git'
```

找到stackoverflow上的一个答案

[https://stackoverflow.com/questions/28983842/remote-rejected-shallow-update-not-allowed-after-changing-git-remote-url](https://stackoverflow.com/questions/28983842/remote-rejected-shallow-update-not-allowed-after-changing-git-remote-url)

那么问题也就很明显了，我在clone原仓库时用了`git clone --depth 1`，导致本地为shallow repo。解决方法也很简单

```bash
git fetch --unshallow origin
```

`origin`就是原仓库。fetch完就能正常push了。