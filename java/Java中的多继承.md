### Java 为什么不支持多继承？

作者：RednaxelaFX
链接：https://www.zhihu.com/question/24317891/answer/65097560
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 1. Java 8 之前

先从 Java 8 之前说起。要区分“声明多继承”与“实现多继承”。

Java 是不允许“实现多继承”，简称不允许“多继承”。但是 Java 支持“声明多继承”——Java 的接口的多继承——一个类可以实现多个接口（“继承”了多个接口上的方法声明），而一个接口可以继承多个接口（同样是“继承”了多个接口上的方法声明）。

接口只允许有方法声明而不允许有实现，因而不会出现像 C++那样的实现多继承的决议问题；抽象类可以有方法实现，但要遵循 Java 类的单继承限制，也避免了实现多继承的问题。这是早期 Java 为了与 C++区分开的一个决定。

至于早期 Java 为何要做这样的设计取舍，不必猜测，看看高司令老人家自己是怎么说的（https://www.artima.com/intv/gosling1.html#part3）：纯粹是高司令老人家的 taste。

### 2. Java 8 时

然后，从 Java 8 开始，接口允许为方法提供“默认实现”了——默认方法（default method）。

因而实质上 Java 8 的接口多继承其实也会涉及到实现多继承，并且语言层面有专门规定去解决实现多继承时选择哪个版本的问题——哪个都不选择，而是在发现会继承多个默认方法实现并且没有 override 时报错，逼使用户显式 override 可能冲突的方法。这使得 Java 8 开始接口可以当作 traits 来使用，达到实现多继承的目的。

### 3. Java 8 语言规范的一些相关新规定

8.4.8. Inheritance, Overriding, and Hiding
https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.4.8

9.4. Method Declarations
https://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.4

13.5.6. Interface Method Declarations
https://docs.oracle.com/javase/specs/jls/se8/html/jls-13.html#jls-13.5.6

### 4. 总结

（1）java 中类不支持多继承，只能单继承，但是可以多实现

（2）java 中接口之间支持多继承，接口可以继承多个继承
