### 1. Maven 添加外部依赖

```xml
 <!--添加外部依赖-->
<dependency>
	<groupId>Ice</groupId>
	<artifactId>Ice</artifactId>
	<version>1.0</version>       
	<scope>system</scope>
	<systemPath>${basedir}/src/lib/Ice.jar</systemPath>
</dependency>

<resources>
	<resource>
		<directory>src/lib</directory>
		<targetPath>BOOT-INF/lib/</targetPath>
		<includes>
	         <include>**/*.jar</include>
		</includes>
	</resource>
	<resource>
		<directory>src/main/resources</directory>
		<targetPath>BOOT-INF/classes/</targetPath>
	</resource>
</resources>

```

### 2. Maven 命令

```
通过maven可以实现按不同环境进行打包部署，命令为:
mvn package -P pro

mvn archetype:generate--构建项目  
mvn clean--项目清理  
mvn test--项目单元测试的编译  
mvn compile--项目源代码的编译  
mvn package--项目打包  
mvn install--发布项目提交到本地仓库  
mvn deploy--发布项目到  
mvn jetty:run--启动jetty容器    
mvn eclipse:clean--清除eclipse的一些系统设置                 
mvn eclipse:eclipse--生成eclipse项目文件
mvn idea:clean--清除idea的一些系统设置                 
mvn idea:idea--生成idea项目文件
mvn dependency:tree--查看依赖树  
mvn assembly:assembly--需要配assembly插件，可用于把指定文件进行打包 tar.gz,zip包
//指定maven参数：  
-DskipTests=true--默认不走单元测试  
-P local--选择资源文件类型 local,需在pom开启资源配置


mvn install:install-file         //mvn 命令
-Dfile=sojson-demo.jar　         //要添加的包
-DgroupId=com.sojson 　　　　　　//pom文件对应的groupId
-DartifactId=com.sojson.demo    //pom文件对应得artifactId
-Dversion=1.0　　　　　　　　　 //添加包的版本
-Dpackaging=jar

```

### 3. Maven 中 scope 的分类

#### compile
默认值。
表示被依赖项目需要参与当前项目的编译、测试、运行周期，打包的时候通常需要包含进去。

#### test
表示依赖项目仅仅参与测试相关的工作，包括测试代码的编译，执行。
比较典型的如 junit。

#### runntime
表示被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与。
与compile相比，跳过编译而已。
比较常见的如 JSR××× 的实现，对应的 API jar 是 compile 的，具体实现是 runtime 的，compile 只需要知道接口就足够了。
oracle jdbc 驱动架包就是一个很好的例子，一般 scope 为 runntime。
另外 runntime 的依赖通常和 optional 搭配使用，optional 为 true。我可以用A实现，也可以用B实现。

#### provided
provided 意味着打包的时候可以不用包进去，别的设施（Web Container）会提供。
事实上该依赖理论上可以参与编译，测试，运行等周期。
相当于compile，但是在打包阶段做了exclude的动作。

#### system
从参与度来说，也 provided 相同。
不过被依赖项不会从 maven 仓库抓，而是从本地文件系统拿，一定需要配合 systemPath 属性使用。
