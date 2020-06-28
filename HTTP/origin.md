### HTTP 头中的 origin 字段

（1）当一个链接或者 XMLHttpRequest 去请求跨域操作，浏览器事实上的确向目标服务器发起了连接请求，并且携带这 origin。
（2）当服务器返回时，浏览器将检查 response 中是否包含 Access-Control-Allow-Origin 字段，当缺少这个字段时，浏览器将 abort，abort 的意思是不显示，不产生事件，就好像没有请求过，甚至在 network 区域里面都看不到。
（3）当存在 origin 这个 header 时，浏览器将检查当前请求所在域是否在这个 access-control-allow-origin 所允许的域内，如果是，继续下去，如果不存在，abort！

彻底搞清 referrer 和 origin
https://blog.csdn.net/zdavb/article/details/51161130
