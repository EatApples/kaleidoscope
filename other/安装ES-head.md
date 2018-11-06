# ES-head

## 安装node，设置环境变量

$ vi /etc/profile

> export NODE_HOME=node所在路径

> export PATH=$NODE_HOME/bin:$PATH


## 解压es-head包
可修改 *Gruntfile.js* 文件中的 *hostname* 配置项为本机地址：

```
 connect: {
                        server: {
                                options: {
                                        port: 9100,
                                           hostname: '机器地址',
                                        base: '.',
                                        keepalive: true
                                }
                        }
                }
```

可修改 ***_site/app.js*** 中的 ***(function( app, i18n )*** 中的 ***base_uri*** 变量，修改默认的 es 9200 地址:

> this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "默认地址";

# 启动
> cd /elasticsearch-head/node_modules/grunt/bin

> ./grunt server &

# ES启动问题汇总
## 1
> max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536]

>max number of threads [1024] for user [XX] likely too low, increase to at least [2048]

解决：切换到root用户，编辑limits.conf 添加类似如下内容

> vi /etc/security/limits.conf

添加如下内容:

> soft nofile 655350
>
> hard nofile 655350
>
> soft nproc 65535
>
> hard nproc 65535

## 2
> max number of threads [1024] for user [lish] likely too low, increase to at least [2048]

解决：切换到root用户，进入limits.d目录下修改配置文件。
vi /etc/security/limits.d/90-nproc.conf
修改如下内容：
> soft nproc 1024
#修改为
> soft nproc 2048


## 3
> max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
解决：切换到root用户修改配置sysctl.conf
> vi /etc/sysctl.conf

添加下面配置：
> vm.max_map_count=655360

并执行命令：
>sysctl -p

然后，重新启动elasticsearch，即可启动成功。

## 运行时请使用非root用户
> ./bin/elasticsearch -d
