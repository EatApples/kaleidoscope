### 谈谈 Spring Bean 的生命周期和作用域？

Spring Bean 生命周期比较复杂，可以分为创建和销毁两个过程。
首先，创建 Bean 会经过一系列的步骤，主要包括：

（1）实例化 Bean 对象。
（2）设置 Bean 属性。
（3）如果我们通过各种 Aware 接口声明了依赖关系，则会注入 Bean 对容器基础设施层面的依赖。具体包括 BeanNameAware、BeanFactoryAware 和 ApplicationContextAware，分别会注入 Bean ID、Bean Factory 或者 ApplicationContext。
（4）调用 BeanPostProcessor 的前置初始化方法 postProcessBeforeInitialization。
（5）如果实现了 InitializingBean 接口，则会调用 afterPropertiesSet 方法。
（6）调用 Bean 自身定义的 init 方法。
（7）调用 BeanPostProcessor 的后置初始化方法 postProcessAfterInitialization。
创建过程完毕。

第二，Spring Bean 的销毁过程会依次调用 DisposableBean 的 destroy 方法和 Bean 自身定制的 destroy 方法。

Spring Bean 有五个作用域，其中最基础的有下面两种：
（1）Singleton，这是 Spring 的默认作用域，也就是为每个 IOC 容器创建唯一的一个 Bean 实例。
（2）Prototype，针对每个 getBean 请求，容器都会单独创建一个 Bean 实例。
从 Bean 的特点来看，Prototype 适合有状态的 Bean，而 Singleton 则更适合无状态的情况。另外，使用 Prototype 作用域需要经过仔细思考，毕竟频繁创建和销毁 Bean 是有明显开销的。

如果是 Web 容器，则支持另外三种作用域：
（3）Request，为每个 HTTP 请求创建单独的 Bean 实例。
（4）Session，很显然 Bean 实例的作用域是 Session 范围。
（5）GlobalSession，用于 Portlet 容器，因为每个 Portlet 有单独的 Session，GlobalSession 提供一个全局性的 HTTP Session。
