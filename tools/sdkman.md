# 一，安转 JDK17

sdkman，不仅仅是 Java 的环境，其他环境也能够管理起来！

### 1. 安装 sdkman

（1）从官方网站下载 SDKMAN

```s
curl -s "https://get.sdkman.io" | bash
```

（2）执行环境变量配置脚本

```s
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

（3）输入下面命令查看 SDKMAN 版本信息，以确保 SDKMAN 安装成功

```s
sdk version
```

### 2. 使用 sdkman

```s
Usage: sdk <command> [candidate] [version]
       sdk offline <enable|disable>

   commands:
       install   or i    <candidate> [version] [local-path]
       uninstall or rm   <candidate> <version>
       list      or ls   [candidate]
       use       or u    <candidate> <version>
       config
       default   or d    <candidate> [version]
       home      or h    <candidate> <version>
       env       or e    [init|install|clear]
       current   or c    [candidate]
       upgrade   or ug   [candidate]
       version   or v
       broadcast or b
       help
       offline           [enable|disable]
       selfupdate        [force]
       update
       flush             [archives|tmp|broadcast|metadata|version]

   candidate  :  the SDK to install: groovy, scala, grails, gradle, kotlin, etc.
                 use list command for comprehensive list of candidates
                 eg: $ sdk list
   version    :  where optional, defaults to latest stable if not provided
                 eg: $ sdk install groovy
   local-path :  optional path to an existing local installation
                 eg: $ sdk install groovy 2.4.13-local /opt/groovy-2.4.13


```

（1）看到支持的所有 Java 相关的 sdk

```s
sdk list java
```

（2）下载 JDK17

```s
sdk install java 17-open
```

下载安装和配置环境变量都是全自动的，稍等片刻，然后就会看到 jdk 已经自动切换到 17 了。

（3）切换版本

当然还可以再切回原来的版本，输入

```s
sdk d java  8.0.302-open
```

再看 jdk 版本已经变成了 8。

# 资料来源

#### 1. 推荐两款工具给爱做实验的人

https://mp.weixin.qq.com/s/0nfjbKIiHHGXkQVeYhxkCQ

#### 2. Mac 安装 SDKMAN

https://www.jianshu.com/p/95f94dd82e8c

#### 3. sdkman 官方文档

https://sdkman.io/usage
