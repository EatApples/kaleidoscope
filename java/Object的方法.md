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
（5）类 Object 本身并不实现 Cloneable 接口，因此对类为 Object 的对象调用 clone 方法将导致在运行时抛出异常

注意：所有数组对象 T[] 默认实现了 Cloneable 接口，而且是浅拷贝

关于浅拷贝与深拷贝的区别，我的理解就是，如果该对象内部其他属性的引用是相同的，则是浅拷贝（堆中数据一致）；否则，是深拷贝（堆中数据不同）

《Effective Java》中的2点建议与之相关：
（1）专家级程序员干脆从来不去覆盖 clone 方法，也从来不去调用它（自己新建对象，然后get/set不香么？）
（2）列表代替数组，保护性拷贝（所有数组对象默认浅拷贝，意味着对外暴露元数据）

#### 6. wait
```
public final native void wait(long timeout) throws InterruptedException;
```
目的：使得当前线程等待，直到特定事件发生
约束：
（1）此方法只能由此对象的 monitor 的所有者线程调用
（2）此方法将当前线程置于该对象的等待集中，并放弃对该对象的所有同步声明（即退出临界区），当前线程将不再被调度（即没有CPU时间片了），且处于休眠状态，直到特定事件发生
（3）当前线程被唤醒时，将从该对象的等待集中删除，并可以被调度，然后以通常的方式与任何其他线程竞争来同步这个对象。一旦当前线程重新获得此对象 monitor 的所有权，之前在该对象上的所有同步声明都恢复到“引用”状态——也就是说，恢复到 wait 方法被调用时的状态。 然后当前线程从调用 wait 方法处返回。 因此，从 wait 方法返回时，对象和当前线程的同步状态与调用 wait 方法时完全一样
（4）线程也可以在特定事件之外的情形下被唤醒，即伪唤醒。虽然实践过程中极少发生，但应用程序必须通过测试导致线程被唤醒的条件来防止这种情况，如果条件不满足，则继续等待。 换句话说，等待应该总是以循环的形式出现，比如这个:
```
synchronized (obj) {
    while (条件不成立){
        obj.wait(timeout);
    }
    ... // 根据情况采取适当的行动
}
```
（5）如果当前线程在等待之前或等待期间被任何线程中断，则会引发 InterruptedException。 这个异常不会被抛出，直到该对象的锁状态按照上面所描述的被还原
（6）wait 方法只将当前线程放置到此对象的 wait 集中，也只解锁此对象，如果当前线程还是其他对象的 monitor 的所有者，则在当前线程等待此对象期间，一直保持其他对象的锁定状态（当前线程如果进入了多个对象的临界区，只调用某对象的 wait 方法，则只退出该对象的临界区，其他对象的临界区都没有退出）

特定事件有哪些？
（1）其他线程调用该对象的 notify 方法，而当前线程碰巧被随机选择为被唤醒的线程
（2）其他线程为这个对象调用 notifyAll 方法
（3）其他线程中断当前线程
（4）超时时间已到，自动被唤醒。如果超时时间设置为零，则超时时间不再起作用，当前线程只能由其他方式唤醒


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
目的：唤醒正在等待此对象的 monitor 的所有线程（即调用过此对象 wait 方法的所有线程）
约束：
（1）一次只有一个线程可以拥有对象的 monitor
（2）此方法只能由此对象的 monitor 的所有者线程调用
（3）不管有多少线程等待此对象，一律唤醒全部（有惊群效应）
（4）被唤醒的线程将无法继续，直到当前线程放弃对该对象的锁定（即 monitor 的所有者线程退出临界区）
（5）被唤醒的线程将以通常的方式与任何其他线程竞争来同步这个对象; 例如，被唤醒的线程在成为下一个锁定这个对象的线程时没有任何的特权或劣势

#### 9. finalize
```
protected void finalize() throws Throwable { }
```
目的：对象被内存回收时做清理操作（类似于析构函数）
约束：
（1）当垃圾回收器确定此对象没有更多的引用时，垃圾回收器将调用此对象的 finalize
（2）子类应该覆盖 finalize 方法以释放系统资源或执行其他清理操作
（3）finalize 方法可以执行任何操作，包括将该对象的引用重新提供给其他线程。但是，finalize 通常的目的是在对象被不可避免地回收之前执行清理操作， 如关闭IO连接
（4）Java 语言并不保证调用对象 finalize 方法的是哪个线程，但是可以保证，当 finalize 被调用时，调用 finalize 的线程不会持有任何用户可见的同步锁
（5）如果 finalize 方法抛出未捕获的异常，则忽略该异常，并且 finalize 方法终止
（6）调用对象的 finalize 方法之后，不会采取进一步操作，直到 Java 虚拟机再次确定任何尚未死亡的线程不再能够访问该对象（包括其他准备回收的对象或类可能采取的行动），此时该对象将被回收
（7）对于任何给定的对象，java 虚拟机最多调用 finalize 方法一次