---
title: Hello World
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Install Hexo

命令行输入：
``` bash
$ npm install hexo-cli -g
```
然后输入：
``` bash
$ npm install hexo --save
```
检查hexo是否安装成功：
``` bash
$ hexo -v
```
如果得到下面的结果，表示安装成功：
```
hexo: 3.3.8
hexo-cli: 1.0.3
os: Windows_NT 10.0.10586 win32 x64
http_parser: 2.7.0
node: 6.11.2
v8: 5.1.281.103
uv: 1.11.0
zlib: 1.2.11
ares: 1.10.1-DEV
icu: 58.2
modules: 48
openssl: 1.0.2l
```

## Configure Hexo

创建一个空文件夹，然后输入：
``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
输入：
``` bash
$ hexo g
$ hexo s
```
会提示：
```
INFO Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
表示hexo本地配置完成。

## Hexo和Github

首先安装扩展：
```
$ npm install hexo-deployer-git --save 
```
然后修改_config.yml文件：
```
deploy:
  type: git
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master
```
生成和部署：
```
$ hexo d -g
```

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)
