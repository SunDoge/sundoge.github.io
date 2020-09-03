---
title: aria2无法下载Amazon S3的文件
date: 2020-09-03 15:30:11
tags:
  - aria2
  - Amazon S3
---

今天用[aria2]下载`s3`上的文件时，出现以下错误：

```
09/03 15:34:47 [ERROR] CUID#7 - Download aborted. URI=https://s3.eu-central-1.amazonaws.com/avg-kitti/data_object_image_2.zip
Exception: [AbstractCommand.cc:351] errorCode=1 URI=https://s3.eu-central-1.amazonaws.com/avg-kitti/data_object_image_2.zip
  -> [SocketCore.cc:1018] errorCode=1 SSL/TLS handshake failure: The TLS connection was non-properly terminated.
```

但是用`wget`却一切正常。开始怀疑`s3`限制连接数，但是减少线程数仍然无法下载。最后在一个不相关的[issue #502]里面找到了解决办法。似乎问题是`Async DNS`引起的。加上`--async-dns=false`后问题解决。

暂时不清楚`Async DNS`干了什么，如果以后看源码弄清楚了再补充。



[aria2]: https://aria2.github.io/
[issue #502]: https://github.com/aria2/aria2/issues/502