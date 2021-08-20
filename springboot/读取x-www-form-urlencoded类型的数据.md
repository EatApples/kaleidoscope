### 背景

阿里云支持报警回调接口，文档中写的是 POST 请求，application/x-www-form-urlencoded 类型的数据。

阿里云文档地址：https://help.aliyun.com/document_detail/60714.html?spm=a2c4g.11186623.6.670.d9d1772agF7Uln

需要接收这些指标数据，然后处理。

### 问题

如何接收这些请求参数？

### 方法

使用注解 @RequestParam Map<String, String> params，不仅能解析 body 中的 x-www-form-urlencoded 类型的数据，还能解析 URL 中 XXX=YYY 类型的请求参数数据。

数据都会存储在 Map 中，使用对应的 KEY 就能获取 VALUE。

### 资料来源

标题： springboot 接收 application/x-www-form-urlencoded 类型的请求，获取不到数据
来源：https://www.cnblogs.com/chLxq/p/11821728.html
