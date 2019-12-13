关于 wait 与 notify/notifyAll 方法的语义，可以先看看 monitor 了解一下上下文。

#### 1. equals
```
public boolean equals(Object obj) {
    return (this == obj);
}
```
目的：判断某个对象是否与当前对象具有等价关系。默认是同一对象的多个引用之间等价。重写equals时，必须重写hashCode，以维护hashCode的约束
约束：前三个是等价关系的表述
（1）自反性
（2）对称性
（3）传递性
（4）一致性。如果对象没有修改（equals方法中涉及到的信息不变），多次调用equals，总是返回同一个结果
（5）非空性。对于任何非空引用值 x，x.equals (null)应该返回 false

#### 2. hashCode
```
public native int hashCode();
```
目的：便于哈希对象的使用
约束：
（1）如果对象没有修改（equals方法中涉及到的信息不变），则在应用执行期间内，多次调用hashCode，总是返回同一个结果（同一应用的多次执行之间，不做此约束）
（2）如果equals判断两对象相等，则hashCode返回的结果也相等
（3）两个不equals的对象返回不同的hashCode，对此不做强制约束，但这样做有利于哈希对象的使用

#### 3. getClass
```
public final native Class<?> getClass();
```
目的：返回此对象的运行时类

#### 4. toString
```
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
目的：使用文本表示这个对象，便于人们见文知义

#### 5. clone
```
protected native Object clone() throws CloneNotSupportedException;
```
目的：创建并返回此对象的副本
约束：
（1）x.clone() != x 为 true，表明内存地址不一样
（2）x.clone().getClass() == x.getClass() 为 true，但不做强制要求
（3）x.clone().equals(x) 为 true，也不做强制要求
（4）如果这个对象的类没有实现 Cloneable 接口，那么就会抛出一个 CloneNotSupportedException 
（4）类 Object 本身并不实现 Cloneable 接口，因此对类为 Object 的对象调用 clone 方法将导致在运行时抛出异常

注意：所有数组对象 T[] 默认实现了 Cloneable 接口，而且是浅拷贝

关于浅拷贝与深拷贝的区别，我的理解就是，如果该对象内部其他属性的引用是相同的，则是浅拷贝（堆中数据一致）；否则，是深拷贝（堆中数据不同）

《Effective Java》中的2点建议与之相关：
（1）专家级程序员干脆从来不去覆盖 clone 方法，也从来不去调用它（自己新建对象，然后get/set不香么？）
（2）列表代替数组，保护性拷贝（所有数组对象默认浅拷贝，意味着对对外暴露元数据）

#### 6. wait
```
public final native void wait(long timeout) throws InterruptedException;
```

#### 7. notify
```
public final native void notify();
```
目的：唤醒正在等待此对象的 monitor 的某个线程（即调用过此对象 wait 方法的线程）
约束：
（1）一次只有一个线程可以拥有对象的 monitor
（2）此方法只能由此对象的 monitor 的所有者线程调用
（3）如果有多个线程等待此对象，则从中随机唤醒一个
（4）被唤醒的线程将无法继续，直到当前线程放弃对该对象的锁定（即 monitor 的所有者线程退出临界区）
（5）被唤醒的线程将以通常的方式与任何其他线程竞争来同步这个对象; 例如，被唤醒的线程在成为下一个锁定这个对象的线程时没有任何的特权或劣势

线程如何成为对象 monitor 的所有者？
（1）通过执行该对象的同步实例方法
```
 public synchronized void doSomething() {
 }

```
（2）通过执行该对象同步块
```
 public void doSomething() {
     synchronized (对象) {
     }
 }

 public void doSomething(){
     synchronized (类) {
     }
 }

```
（3）对于 Class 类型的对象，通过执行该类的同步静态方法
```
 public static synchronized void doSomething(){

 }

```
#### 8. notifyAll
```
public final native void notifyAll();
```

#### 9. finalize
```
protected void finalize() throws Throwable { }
```