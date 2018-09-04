### 1 Maven添加外部依赖

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

### 2 Maven命令

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
