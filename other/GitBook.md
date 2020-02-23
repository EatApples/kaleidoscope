## GitBook 相关

### npm 安装 less 报错 rollbackFailedOptional: verb npm-session

（1）更改 npm 源

解决方法：更换成淘宝的源

> npm config set registry https://registry.npm.taobao.org

配置后可通过下面方式来验证是否成功

> npm config get registry

或

> info express

（2）使用 cnpm

先删除原有的所有代理

> npm config rm proxy

> npm config rm https-proxy

然后使用

> npm install -g cnpm --registry=https://registry.npm.taobao.org

安装淘宝的 cnpm，就可以使用淘宝的 cnpm 了

### GitBook 常用的命令

```
gitbook init //初始化目录文件
gitbook help //列出gitbook所有的命令
gitbook --help //输出gitbook-cli的帮助信息
gitbook build //生成静态网页
gitbook serve //生成静态网页并运行服务器
gitbook build --gitbook=2.0.1 //生成时指定gitbook的版本, 本地没有会先下载
gitbook ls //列出本地所有的gitbook版本
gitbook ls-remote //列出远程可用的gitbook版本
gitbook fetch 标签/版本号 //安装对应的gitbook版本
gitbook update //更新到gitbook的最新版本
gitbook uninstall 2.0.1 //卸载对应的gitbook版本
gitbook build --log=debug //指定log的级别
gitbook builid --debug //输出错误信息
```

### 扩展阅读

#### 1. GitBook 简明教程

http://www.chengweiyang.cn/gitbook/index.html
