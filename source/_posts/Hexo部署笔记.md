---
title: Hexo部署笔记
date: 2017-03-01 16:10:54
tags:
---

# 主题选择

- [hexo-theme-apollo](https://github.com/pinggod/hexo-theme-apollo "apollo")
- 网上找了个水滴的图标，好看，一点点寓意。

# 多客户端写文章的问题

`hexo d` 执行后 *hexo* 将 `hexo g` 生成的 *public* 文件夹下的静态文件部署到目标服务器，其它 *hexo* 环境相关的内容则还在本机上，这样势必导致之前辛苦装的一系列插件，主题之类的东西没有办法同步到其它机器上，更换电脑是件麻烦事。

暂且想到集中解决方案
- 使用 dropbox，OneDriver 之类的文件同步服务同步源文件
- 另外建一个repo同步源文件
- 网络上还有一种将源文件提交到分支的做法
- 