### 1. 日期显示
```sh
date +"%Y%m%d%H%M%S"

%H 小时，24小时制（00~23）
%I 小时，12小时制（01~12）
%k 小时，24小时制（0~23）
```

### 2. 设置文本格式
```
set fileformats=unix
```

### 3. CURL
+ get请求
```sh
curl "http://www.baidu.com"     如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地
curl -i "http://www.baidu.com"  显示全部信息
curl -l "http://www.baidu.com"  只显示头部信息
curl -v "http://www.baidu.com"  显示get请求全过程解析
wget "http://www.baidu.com"     也可以
```

+ post请求
```sh
curl -d "param1=value1&param2=value2" "http://www.baidu.com"
```

+ json格式的post请求
```sh
curl -l -H "Content-type: application/json" -X POST -d '{"phone":"13521389587","password":"test"}' http://domain/apis/users.json
```
