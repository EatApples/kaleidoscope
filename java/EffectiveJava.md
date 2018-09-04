#### 1.	可见性与原子性

#### 2.	volatile

#### 3.	线程安全性的几种级别
+ 3.1	不可变 immutable
+ 3.2	无条件的线程安全
+ 3.3	有条件的线程安全
+ 3.4	非线程安全
+ 3.5	线程对立 thread-hostile

#### 4.	延迟初始化
+ 4.1	双重校验加锁
```java
		private volatile FieldType field;
		FieldType getField(){
			FieldType result =  field;
			if(result == null){
				synchronized(this){
					result=field;
					if(result == null){
						field = result = computeFieldValue();
					}

				}
			}
			return result;
		}
```		

可以用 `volatile` 实现一个双重检查锁的单例模式：
```java
public class Singleton {

    private static volatile Singleton singleton;

    private Singleton() {

    }

    public static Singleton getInstance() {

        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }

}
```
这里的 `volatile` 关键字主要是为了防止指令重排。
如果不用 `volatile` ，`singleton = new Singleton();`，这段代码其实是分为三步：

- 分配内存空间。(1)
- 初始化对象。(2)
- 将 `singleton` 对象指向分配的内存地址。(3)

加上 `volatile` 是为了让以上的三步操作顺序执行，反之有可能第二步在第三步之前被执行就有可能某个线程拿到的单例对象是还没有初始化的，以致于报错。

+ 4.2	 内部类
```java
		private static class FieldHolder{
			static final FieldType field = computeFieldValue();
		}
		static FieldType getFild(){ return FieldHolder.field; }
```
+ 4.3	单重检查模式，条件：可以接受可重复初始化的域
```java
		private volatile FieldType field;
		FieldType getField(){
			FieldType result =  field;

			if(result == null){
				field = result = computeFieldValue();
			}

			return result;
		}
```
#### 5.	 for vs while 变量局部性胜出
```java
		for(int i=0, n = expensiveComputation(); i<n; i++){
			doSomething();
		}
```
#### 6.	精确答案，不要 double，用 BigDecimal

#### 7.	装拆箱的时候注意 ==

#### 8.	随机数用 Random.nextInt(int)

#### 9.	内部类不需要与外部相关联，声明为静态的 static

#### 10.	通过接口引用对象

#### 11.	重写 equals 方法
 ```java
 	    @Override
		public boolean equals(Object obj) {
			if (this == obj)
				return true;
			if (obj instanceof Pair) {
				Pair p = (Pair) obj;
				return (this.queueName == p.queueName && this.exchangeName == p.exchangeName);
			}
			return false;
		}
```		
#### 12.	覆盖 equals 时总要覆盖 hashCode

#### 13.	专家级程序员干脆从来不去覆盖 clone 方法，也从来不去调用它

#### 14.	考虑实现 comparable 接口
```java
		public interface Comparable<T> {
			int compareTo(T t);
		}
```
#### 15.	单例
```java
		// Singleton with public final field
		public class Elvis{
			public static final Elvis INSTANCE = new Elvis();
			private Elvis(){ ...}

			public void leaveTheBuilding(){ ...}
		}

		//Singleton with static factory
		public class Elvis{
			private static final Elvis INSTANCE = new Elvis();
			private Elvis(){ ...}
			public static Elvis getInstance(){ return INSTANCE;}

			public void leaveTheBuilding(){ ...}
		}

		//Enum singleton- the preferred approach
		public enum Elvis{
			INSTANCE;

			public void leaveTheBuilding(){ ...}
		}
```
#### 16.	只要类是自己管理内存，程序员就应该警惕内存泄漏问题：清除过期引用

#### 17.	类具有公有的静态final数组域，或者返回这种域的访问方法，这几乎总是错误的
**公有类永远都不应该暴露可变的域，使用访问方法！**

#### 18.	String -> StringBuilder; BigInteger -> BitSet

#### 19.	复合（装饰类）优先于继承（明确具有 is-a 关系）

#### 20.	列表代替数组，保护性拷贝

#### 21.	Enum 感觉很犀利

#### 22.	抛有意义的异常

```java
		IllegalArgumentException		         参数值不正确
		IllegalStateException			           对象状态不合适
		NullPointerException			           空指针
		IndexOutOfBoundsException		         数组越界
		ConcurrentModificationException		   并发修改
		UnsupportedOperationException		     不支持方法
```
