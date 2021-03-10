### EL 表达式

EL 是 JSP 2.0 增加的技术规范，其全称是表达式语言(Expression Language)。

EL 语言的灵感来自于 ECMAScript 和 XPath 表达式语言。EL 表达式语言是一种简单的语言，提供了在 JSP 中简化表达式的方法，目的是为了尽量减少 JSP 页面中的 Java 代码，使得 JSP 页面的处理程序编写起来更加简洁，便于开发和维护。

在 JSP 中访问模型对象是通过 EL 表达式的语法来表达。所有 EL 表达式的格式都是以“${}”表示。例如，${userinfo} 代表获取变量 userinfo 的值。当 EL 表达式中的变量不给定范围时，则默认在 page 范围查找，然后依次在 request、session、application 范围查找。也可以用范围作为前缀表示属于哪个范围的变量，例如：`${ pageScope.userinfo}`表示访问 page 范围中的 userinfo 变量。

#### JSP 表达式语言

https://www.runoob.com/jsp/jsp-expression-language.html
