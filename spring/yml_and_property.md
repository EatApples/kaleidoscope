### 1. 双引号

不会转义字符串里面的特殊字符，特殊字符会作为本身想表示的意思

```yml
name: "zhangsan \n lisi"
```

输出：

```
zhangsan
lisi
```

### 2. 单引号

会转义特殊字符，特殊字符最终只是一个普通的字符串数据

```yml
name: ‘zhangsan \n lisi'
```

输出：

```
zhangsan \n lisi
```

### 3. 对象、Map（属性和值）（键值对）

例如配置类中的字段为

```java
Map<String,String> maps;
```

在 yml 配置文件中，行内写法

```yml
person.maps: { key1: value1, key2: value2 }
```

需要注意:号后的空格，或者

```yml
person:
  maps:
  key: value
```

在 properties 配置文件中

```yml
person.maps.key=value
```

### 4. 数组（List、Set）

在 yml 配置文件中

```yml
person:
  list:
    - 1
    - 2
    - 3
```

行内写法

```yml
person:
  list: [1, 2, 3]
```

在 properties 配置文件中

```yml
person.list[0]=1
person.list[1]=2
person.list[2]=3
```

### 资料来源

#### 1. 史上最全面的 Spring Boot 配置文件深入讲解

http://www.manongjc.com/article/20519.html
