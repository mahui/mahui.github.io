# mahui.github.io 源码

- 安装hexo
```
npm install -g hexo-cli
```
- clone 源码
```
git clone git@github.com:mahui/mahui.github.io.git
cd mahui.github.io
git checkout source
```

以上为新机需要执行的初始化环境的操作

---------------------------

- 启动hexo
```
hexo s -p $port
```
- 新建文章
```
hexo new [$layout] <$title>
```
- 生成静态文件
```
hexo g
```
- 部署到服务器
```
hexo d
```