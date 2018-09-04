### 问题
同一个应用，在`Tomcat`中启动，加载了两次。

在容器中部署时，也出现过两次加载的情况。

### 原因
`server.xml`配置片段为：
```xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
    <Context path="/XXX-YYY" reloadable="false"  docBase="/app/tomcat/webapps/ROOT"/>
</Host>
```

解释：

`Host`：默认情况`Host`节点下是没有`Context`节点的，如果需要多站点，就必须添加`Context`指定`Web`应用的文件路径。

`appBase`：是指定虚拟主机的目录，可以指定绝对目录，也可以指定相对于`Tomcat`根目录的相对目录。如果没有此项，默认为`webapps`。这个目录下存放的`Web`应用会由该虚拟主机自动管理。

`path`：指定访问该`Web`应用的`URL`入口。

`reloadable`：如果这个属性设为`true`，`Tomcat`服务器在运行状态下会监视在`WEB-INF/classes`和`WEB-INF/lib`目录下`class`文件的改动，如果监测到有`class`文件被更新的，服务器会自动重新加载`Web`应用。

`docBase`：指定`Web`应用的文件路径。可以给定绝对路径，也可以给定相对于`Host`的`appBase`属性的相对路径。如果`Web`应用采用开放目录结构，那就指定`Web`应用的根目录；如果`Web`应用是个`war`文件，那就指定`war`文件的路径。

因为：

（1）应用本来就放在`Tomcat`的默认`webapps`目录下，`Tomcat`在启动时肯定会`加载1次`

（2）然后又在`server.xml`中做了配置，这样`Tomcat`就又`加载1次`

结果，`Tomcat`就会加载两次。

### 解决方案
将`server.xml`配置中`appBase`的值`webapps`改为`/`；

或者在`Tomcat`根目录下建一个`webapps1`文件夹，改成`webapps1`也行。

### 扩展阅读
#### 1. Tomcat-在发布项目时两次重复加载的问题介绍与解决
https://www.cnblogs.com/hwaggLee/p/5151827.html
