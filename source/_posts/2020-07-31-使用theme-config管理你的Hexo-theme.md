---
title: 使用theme_config管理你的Hexo theme
date: 2020-07-31 01:47:42
tags: Hexo
---

自Hexo 2.8.2之后[[1]][hexo doc]，Hexo支持使用`theme_config`来配置theme。在这之前，如果我们想要使用git来更新我们theme，同时又想用git管理theme的config，我所知道的唯一一个办法，就是fork一份theme，修改其中的`_config.yml`，再用`git submodule`将fork的theme引入博客仓库中。这个过程比较繁琐。

现在，我们可以利用博客仓库下的`_config.yml`来覆盖theme仓库下的config。以我使用的NexT为例，默认的Muse scheme并不和我意，我希望将其修改成Mist scheme，只需要添加几行配置：

```yaml
theme_config:
  scheme: Mist
```

Hexo 5.0.0之后[[1]][hexo doc]，我们还可以为每个theme创建一个单独的config文件来进行管理，文件命名规则为`_config.[theme].yml`。还是以NexT为例，创建`_config.next.yml`，修改内容为：

```yaml
scheme: Mist
```

theme就配置好了。

很奇怪的一点是，目前很多关于theme的资料都没有提到Hexo已经支持覆盖theme config，导致我使用了很长一段时间的Hexo 3.9，却还是在用fork+submodule的方法来管理theme。

下次我将尝试使用github action来自动deploy我的博客。

***

2020-07-31 Update

Hexo 5.0.0支持使用npm配置theme了。以后不需要手动管理版本，只需要

```
npm i hexo-theme-next --save
```

目前通过 npm 安装的主题在执行 hexo clean 时会报错，是 Hexo 的已知问题，将在下个版本修复。

[hexo doc]: https://hexo.io/docs/configuration 