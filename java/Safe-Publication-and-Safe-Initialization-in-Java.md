### Safe Publication and Safe Initialization in Java
原文链接：https://shipilev.net/blog/2014/safe-public-construction/

部分摘要如下：

Two of these constructions are "`Safe Publication`" and "`Safe Initialization`". We will take a look at Singletons to explain both constructions. It is important to read these singleton examples as a tutorial in Java concurrency, and not as a promotion for this particular design pattern.

这里通过单例模式来解释安全发布与安全初始化。

A reasonable SingletonFactory has a few properties:

合理的单例工厂有几个性质:

1. It provides the public API for getting a Singleton instance.（提供获得单例的公有接口）

2. It is thread-safe. No matter how many threads are requesting a Singleton, all threads will get the same Singleton instance, regardless of the current state.（线程安全）

3. It is lazy. One can argue about this, but non-lazy factories are not interesting for our discussion. Singleton initialization should happen with the first request for a Singleton, not when Singleton class is initialized. If no one wants a Singleton instance, it should not be instantiated.（延迟加载（非必须））

4. It is efficient. The overheads for managing a Singleton state should be kept at minimum.（高效（非必须））

### SynchronizedCLFactory
…​as it has the properties (1), (2), and (3); but lacks property (4), since we do synchronization on every access for get().

非高效

```java
public class SynchronizedCLFactory {
  private Singleton instance;

  public Singleton get() {
    synchronized (this) {
      if (instance == null) {
        instance = new Singleton();
      }
      return instance;
    }
  }
}
```

### UnsafeDCLFactory
双重校验加锁（非安全发布）

Alas, this construction does not work properly for two reasons.

One could think that after "check 1" had succeeded, the Singleton instance is properly initialized, and we can return it. That is not correct: the Singleton contents are only fully visible to the constructing thread! There are no guarantees that you will see Singleton contents properly in other threads, because you are racing against the initializer thread. Once again, even if you have observed the non-null instance, it does not mean you observe its internal state properly. In JMM-ese speak, there is no happens-before between the initializing stores in Singleton constructor, and the reads of Singleton fields.

构造与发布之间没有 happens-before 规则，不是安全发布

Notice that we do several reads of instance in this code, and at least "read 1" and "read 3" are the reads without any synchronization — that is, those reads are racy. One of the intents of the Java Memory Model is to allow reorderings for ordinary reads, otherwise the performance costs would be prohibitive. Specification-wise, as mentioned in happens-before consistency rules, a read action can observe the unordered write via the race. This is decided for each read action, regardless what other actions have already read the same location. In our example, that means that even though "read 1" could read non-null instance, the code then moves on to returning it, then it does another racy read, and it can read a null instance, which would be returned!

读操作之间重排，可能读到 null

```java
public class UnsafeDCLFactory {
  private Singleton instance;

  public Singleton get() {
    if (instance == null) {  // read 1, check 1
      synchronized (this) {
        if (instance == null) { // read 2, check 2
          instance = new Singleton();
        }
      }
    }
    return instance; // read 3
  }
}
```

### Safe Publication
Now we are going to describe the concept of safe publication. Safe publication differs from a regular publication on one crucial point. Safe publication makes all the values written before the publication visible to all readers that observed the published object. It is a great simplification over the JMM rules of engagement with regards to actions, orders and such.

安全发布意味着对象在读操作（对所有读操作可见）之前，所有数据已经写入。

There are a few trivial ways to achieve safe publication:

满足安全发布的途径：

1. Exchange the reference through a properly locked field (JLS 17.4.5)（在锁区域更改引用）

2. Use static initializer to do the initializing stores (JLS 12.4)（使用静态初始化块进行初始化）

3. Exchange the reference via a volatile field (JLS 17.4.5), or as the consequence of this rule, via the AtomicX classes（通过 volatile 字段更改引用，或者根据这个规则，通过原子类更改引用）

4. Initialize the value into a final field (JLS 17.5).（final字段的初始化）

### HolderFactory
Classic holder idiom does roughly the same, but piggy-backs on class initialization locks. It safely publishes the object by doing the initialization in the static initializer. Note this thing is lazy, since we do not initialize Holder until the first call to get():

延迟加载
```java
public class HolderFactory {
  public static Singleton get() {
    return Holder.instance;
  }

  private static class Holder {
    public static final Singleton instance = new Singleton();
  }
}
```

### SafeDCLFactory
Now, get back to the infamous volatile DCL, that works because now we safely publish the singleton via volatile:

双重校验加锁现在是安全发布了
```java
public class SafeDCLFactory {
  private volatile Singleton instance;

  public Singleton get() {
    if (instance == null) {  // check 1
      synchronized(this) {
        if (instance == null) { // check 2
          instance = new Singleton();
        }
      }
    }
    return instance;
  }
}
```

### FinalWrapperFactory

There is another way to provide the same guarantees: final fields. Since it is already too late to write to final field outside of constructor, we have to do a wrapper.

使用 final 也能达到同样的效果
```Java
public class FinalWrapperFactory {
  private FinalWrapper wrapper;

  public Singleton get() {
    FinalWrapper w = wrapper;
    if (w == null) { // check 1
      synchronized(this) {
        w = wrapper;
        if (w == null) { // check2
          w = new FinalWrapper(new Singleton());
          wrapper = w;
        }
      }
    }
    return w.instance;
  }

  private static class FinalWrapper {
    public final Singleton instance;
    public FinalWrapper(Singleton instance) {
      this.instance = instance;
    }
  }
}
```

### Safe Initialization
Moving on. Java allows us to declare an object in a manner always safe for publication, that is, provides us with an opportunity for safe initialization. Safe initialization makes all values initialized in constructor visible to all readers that observed the object, regardless of whether the object was safely published or not. Java Memory Model guarantees this if all the fields in object are final and there is no leakage of the under-initialized object from constructor. This is an example of safe initialization:

安全初始化意味着对象在读操作（对所有读操作可见）之前，所有数据已经在构造器中初始化，无论对象是否安全发布。

```java
public class SafeSingleton implements Singleton {
  final Object obj1;
  final Object obj2;
  final Object obj3;
  final Object obj4;

  public SafeSingleton() {
    obj1 = new Object();
    obj2 = new Object();
    obj3 = new Object();
    obj4 = new Object();
  }
}
```
