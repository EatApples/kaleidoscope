### 1. 第一步，项目中添加依赖

（1）添加 组件 依赖

目标项目的 dependencies 中，添加如下依赖：

```xml

<dependencies>
    <!--自动生成测试代码-->
    <!-- https://mvnrepository.com/artifact/eu.stamp-project/evosuite-standalone-runtime -->
    <dependency>
        <groupId>eu.stamp-project</groupId>
        <artifactId>evosuite-standalone-runtime</artifactId>
        <version>1.1.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <!-- 默认的版本为3.8.1，修改为4.x,因为3.x使用的为编程的方式，4.x为注解的形式。 -->
        <version>4.12</version>
        <!-- 表示这个依赖的作用域范围为test -->
        <scope>test</scope>
    </dependency>
</dependencies>
```

（2）添加插件依赖

项目的 plugins 中，添加如下依赖：

```xml
<!-- 单元测试插件 -->
<build>
    <plugins>
        <!--自动生成测试代码插件-->
        <plugin>
            <groupId>org.evosuite.plugins</groupId>
            <artifactId>evosuite-maven-plugin</artifactId>
            <version>1.0.6</version>
        </plugin>
    </plugins>
</build>
```

### 2. 第二步，使用指令生成测试代码

进入 pom 所在文件路径，执行以下指令：

（1）全量生成单测代码

```s
mvn clean compile -DmemoryInMB=4000 -Dcores=8 evosuite:generate

```

（2）生成指定类的单测代码

```s

mvn clean compile -DmemoryInMB=2000 -Dcores=2 -Dcuts=完整包类名1,完整包类名2 evosuite:generate

```

简单说明下：

```s
compile 表示编译。evosuite 是基于编译后的 .class 文件生成用例的，所以需要先编译。
-DmemoryInMB=2000 表示使用 2000MB 的内存
-Dcores=2 表示用 2 个 cpu 来并行加快生成速度
-Dcuts=alexp.blog.service.PostServiceImpl 表示只针对 alexp.blog.service.PostServiceImpl 这个类生成用例。多个用例可以用英文逗号分隔
-DtargetFolder=src/test/java/evosuite 表示生成的用例放到 src/test/java/evosuite 。
evosuite:generate 表示执行生成用例
evosuite:export 表示导出用例到 targetFolder 的值所在的目录中
```

### 3. 第三步，调整测试代码，验证覆盖率

（1）将生成的单测代码（如果不指定路径，默认在项目路径下，名为：.evosuite/best-tests）拷贝到 src/test/java 中

（2）执行单测，一般会发现覆盖率为 0%（不要慌）。要是有覆盖率，后面就不用看了

（3）生成的文件一般成对出现

```s
一个名为：[YOUR_CLASS_NAME]_ESTest_scaffolding.java
一个名为：[YOUR_CLASS_NAME]_ESTest.java
```

示例：

```java
MessageSourceAutoConfiguration_ESTest_scaffolding.java
MessageSourceAutoConfiguration_ESTest.java
```

（4）将`[YOUR_CLASS_NAME]_ESTest_scaffolding.java` 通通删除（整个类都是 mock，不会执行你的真正的代码的）

（5）将`[YOUR_CLASS_NAME]_ESTest.java`调整一下

```java
//第3步，清除无效的引用
import org.junit.Test;
import static org.junit.Assert.*;
import static org.evosuite.runtime.EvoAssertions.*;
import java.nio.charset.Charset;
import org.evosuite.runtime.EvoRunner;
import org.evosuite.runtime.EvoRunnerParameters;
import org.evosuite.runtime.javaee.injection.Injector;
import org.junit.runner.RunWith;
import org.springframework.context.support.AbstractResourceBasedMessageSource;
//第2步，清理类上的注解（通通不要）
@RunWith(EvoRunner.class) @EvoRunnerParameters(mockJVMNonDeterminism = true, useVFS = true, useVNET = true, resetStaticState = true, separateClassLoader = true, useJEE = true)
//第1步，去除 extends（因为上一步已经把这个scaffolding类删除了）
public class MessageSourceAutoConfiguration_ESTest extends MessageSourceAutoConfiguration_ESTest_scaffolding {
```

（6）运行单测，解决 failure 和 error，完成！

# 资料来源

#### 1. 单测代码自动生产工具

http://testerhome.com/topics/17133

#### 2. 官方文档

https://www.evosuite.org/
