### 读取 SpringBoot 项目内的文件

（1）读取 /src/main/resources/ 下的文件

```java
ClassPathResource classPathResource = new ClassPathResource(fileName);
```

（2）读取 SpringBoot 的 jar 包所在路径

```java
ApplicationHome h = new ApplicationHome(Applicaiton.class);
String dirPath = h.getSource().getParentFile().toString();
```

#### SpringBoot 打成 jar 运行后无法读取 resources 里的文件

https://blog.csdn.net/zhuyu19911016520/article/details/98071242
