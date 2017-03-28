---
title: Hexo部署笔记
date: 2017-03-01 16:10:54
categories:
- 笔记
tags:
- hexo
- 部署文档
---

# 主题选择

- [hexo-theme-apollo](https://github.com/pinggod/hexo-theme-apollo "apollo")
- 网上找了个水滴的图标，好看，一点点寓意。

# 多客户端写文章的问题

`hexo d` 执行后 *hexo* 将 `hexo g` 生成的 *public* 文件夹下的静态文件部署到目标服务器，其它 *hexo* 环境相关的内容则还在本机上，这样势必导致之前辛苦装的一系列插件，主题之类的东西没有办法同步到其它机器上，更换电脑是件麻烦事。

暂且想到集中解决方案
- 使用 dropbox，OneDriver 之类的文件同步服务同步源文件
- 另外建一个repo同步源文件
- 网络上还有一种将源文件提交到新分支的做法

看了`hexo init` 后的 *blog_dir* ,显然 *hexo* 已经考虑到这方面的问题，支持将源文件提交到 *git* 仓库中，因为 *.gitignore* 已经初始化好了。

作为一个稍微有点儿强迫症的狮子座，决定采用新建分支的方法

``` shell
cd $blog_dir
git init
git add *
git checkout -b $branch_name
git remote add origin git@github.com:xxxx/xxxx.github.io.git
git push origin $branch_name:$branch_name
```
需要注意的是，*_config.yml* 中 deploy 节点不要暴露敏感信息。
在新的客户端，只需要将代码clone下来，`checkout`到相应分支，`npm install -g hexo-cli`,`npm install` 之后 `new`, `generate`, `deploy` 即可

关于源码的维护，可以将当前 repo 的 default branch 设置为源码的分支，以方便后续维护。
