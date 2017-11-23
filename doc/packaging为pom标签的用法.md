# <packaging>pom</packaging>的用法
## （1）聚合（多模块）
意义：一次构建所有想要构建的项目

```
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.son.nexus</groupId>
    <artifactId>nexus-indexer</artifactId>
    <version>2.0.0</version>
    <packaging>pom</packaging>//本身也是一个maven工程
    <modules>
          <module>account-email</module>//想要构建的项目，这里写的是当前pom文件下的相对路径地址
        <module>account-persilist</module>
    </modules>
</project>
```

聚合pom文件的packaging标签一定要是pom，其工程就只是一个pom文件，没有其他的实现代码。一般来说模块处的目录名应与其artifactId一致。聚合模块与其他模块的目录结构并非一定要父子关系。

## （2）继承
### 父pom
```
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.son.nexus</groupId>
    <artifactId>nexus-indexer</artifactId>
    <version>2.0.0</version>
    <packaging>pom</packaging>//本身也是一个maven工程
     <dependencies>
        <dependency>
            <groupId>com.juv</groupId>
            <artifactId>project-B</artifactId>
            <version>1.0.0</version>
            <exclusions>
                <exclusion>
                    <groupId>com.juv</groupId>
                    <artifactId>project-C</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.juv</groupId>
            <artifactId>project-B</artifactId>
            <version>1.1.0</version>
        </dependency>
    </dependencies>
</project>
```

父pom的packaging也是pom

### 子pom
```
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.son.nexus</groupId>
    <artifactId>nexus-B</artifactId>
    <version>2.0.0</version>
    <packaging>jar</packaging>
    <parent>
        <groupId>org.son.nexus</groupId>
        <artifactId>nexus-C</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>//相对路径
    </parent>
</project>  
```

子pom的packaging则不一定要是pom，但一定有parent标签。子类的groupId和version也可以继承于父pom文件。