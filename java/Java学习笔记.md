# 1. Collections类和Collection接口
## 1.1 Collection
（1）Collection是最基本的集合接口，一个Collection代表一组Object，即Collection的元素（Elements）。子接口包括 List和 Set。

（2）所有实现 Collection 接口的类都必须提供两个标准的构造函数：无参数的构造函数用于创建一个空的 Collection ，有一个 Collection 参数的构造函数用于创建一个新的 Collection ，这个新的 Collection与传入的 Collection 有相同的元素。后一个构造函数允许用户复制一个 Collection。

（3）如何遍历 Collection 中的每一个元素？不论 Collection 的实际类型如何，它都支持一个 iterator() 的方法，该方法返回一个迭代子，使用该迭代子即可逐一访问 Collection 中每一个元素。典型的用法如下：
```java
Iterator it = collection.iterator(); // 获得一个迭代子  
　　while(it.hasNext()) {  
　　　Object obj = it.next(); // 得到下一个元素  
}  
```

## 1.2 Collections
Collections是针对集合类的一个帮助类，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作

# 2. Set、Map、List的区别
## 2.1 Set 	
（1） 无序，值不重复。

（2）只关心某个元素是否属于Set，而不关心它的顺序--否则应该使用List。

（3）只能通过游标来取值。

### 2.1.1 HashSet
为快速查找设计的Set。存入HashSet的对象必须定义hashCode()。

### 2.1.2 TreeSet
保存次序的Set, 底层为树结构。使用它可以从Set中提取有序的序列。

### 2.1.3 LinkedHashSet
 具有HashSet的查询速度，且内部使用链表维护元素的顺序（插入的次序）。于是在使用迭代器遍历Set时，结果会按元素插入的次序显示。

## 2.2 List
（1）有序，值允许重复。

（2）List 按对象进入的顺序保存对象，不做排序或编辑操作。

（3）List 可以通过下标来取得值。

（4）ArrayList，Vector， LinkedList 是 List 的实现类。ArrayList 是线程不安全的， Vector 是线程安全的，这两个类底层都是由数组实现的；LinkedList 是线程不安全的，底层是由链表实现的。

### 2.2.1 ArrayList类
（1）ArrayList实现了可变大小的数组。它允许所有元素，包括null。ArrayList没有同步。

（2）size，isEmpty，get，set方法运行时间为常数。但是add方法开销为分摊的常数，添加n个元素需要O(n)的时间。其他的方法运行时间为线性。

（3）每个ArrayList实例都有一个容量（Capacity），即用于存储元素的数组的大小。这个容量可随着不断添加新元素而自动增加，但是增长算法 并没有定义。当需要插入大量元素时，在插入前可以调用ensureCapacity方法来增加ArrayList的容量以提高插入效率。

### 2.2.2 Vector类
（1）Vector非常类似ArrayList，但是Vector是同步的。

（2）由Vector创建的Iterator，虽然和ArrayList创建的Iterator是同一接口，但是，因为Vector是同步的，当一个Iterator被创建而且正在被使用，另一个线程改变了Vector的状态（例如，添加或删除了一些元素），这时调用Iterator的方法时将抛出ConcurrentModificationException，因此必须捕获该异常。

（3）Stack继承自Vector，实现一个后进先出的堆栈。Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop方法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置。Stack刚创建后是空栈。

### 2.2.3 LinkedList类
（1）LinkedList实现了List接口，允许null元素。

（2）LinkedList提供额外的get，remove，insert方法在 LinkedList的首部或尾部。如下列方法：addFirst(), addLast(), getFirst(), getLast(), removeFirst()和 removeLast(), 这些方法 （没有在任何接口或基类中定义过）。这些操作使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。

（3）注意LinkedList没有同步方法。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List：
```
	List list = Collections.synchronizedList(new LinkedList(...));
```

## 2.3 Map		
（1）成对的数据结构，健值必须具有唯一性（键不能同，否则值替换）。

（2）Map同样对每个元素保存一份，但这是基于"键"的，Map也有内置的排序，因而不关心元素添加的顺序。如果添加元素的顺序对你很重要，应该使用 LinkedHashMap。

（3）Map 是键值对集合，HashTable 和 HashMap 是 Map 的实现类。HashTable 是线程安全的，不能存储 null 值；HashMap 不是线程安全的，可以存储 null 值。

### 2.3.1 Hashtable类
（1）Hashtable继承Map接口，实现一个key-value映射的哈希表。任何非空（non-null）的对象都可作为key或者value。

（2）Hashtable是同步的。添加数据使用 put(key, value) ，取出数据使用get(key)，这两个基本操作的时间开销为常数。

（3）Hashtable 通过初始化容量 (initial capacity) 和负载因子 (load factor) 两个参数调整性能。通常缺省的 load factor0.75 较好地实现了时间和空间的均衡。增大 load factor 可以节省空间但相应的查找时间将增大，这会影响像get和 put 这样的操作。

（4）由于作为 key 的对象将通过计算其散列函数来确定与之对应的 value 的位置，因此任何作为 key 的对象都必须实现 hashCode 方法和 equals 方法。 hashCode 方法和 equals 方法继承自根类 Object ，如果你用自定义的类当作 key 的话，要相当小心，按照散列函数的定义，如果两个对象相同，即 obj1.equals(obj2)==true，则它们的 hashCode 必须相同，但如果两个对象不同，则它们的 hashCode 不一定不同，如果两个不同对象的 hashCode 相同，这种现象称为冲突，冲突会导致操作哈希表的时间开销增大，所以尽量定义好的 hashCode() 方法，能加快哈希表的操作。

（5）如果相同的对象有不同的 hashCode ，对哈希表的操作会出现意想不到的结果（期待的 get 方法返回null），要避免这种问题，只需要牢记一条：要同时复写 equals 方法和 hashCode 方法，而不要只写其中一个。

### 2.3.2 HashMap类
（1）HashMap和Hashtable类似，也是基于hash散列表的实现。不同之处在于HashMap是非同步的，并且允许null，即null value和null key。

（2）将HashMap视为Collection时 （values()方法可返回Collection），其迭代子操作时间开销和HashMap的容量成比例。因此，如果迭代操作的性能相当重要的话，不要将HashMap的初始化容量设得过高，或者load factor过低。

（3）LinkedHashMap类：类似于 HashMap，但是迭代遍历它时，取得“键值对”的顺序是其插入次序，或者是最近最少使用 (LRU) 的次序。只比 HashMap慢一点。而在迭代访问时发而更快，因为它使用链表维护内部次序。

### 2.3.3 WeakHashMap类 （弱键（ weak key ））
WeakHashMap是一种改进的HashMap，它是为解决特殊问题设计的，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收。

### 2.3.4 TreeMap 类
（1）基于红黑树数据结构的实现。

（2）查看“键”或“键值对”时，它们会被排序 (次序由 Comparable 或 Comparator 决定 )。TreeMap 的特点在于，你得到的结果是经过排序的。

（3）TreeMap带有 subMap()方法，它可以返回一个子树。

### 2.3.5 IdentifyHashMap 类
使用 == 代替 equals()对“键”作比较的 hash map。专为解决特殊问题而设计。


# 3. 容器类和Array的区别、择取
（1）容器类仅能持有对象引用（指向对象的指针），而不是将对象信息copy一份至数列某位置。

（2）一旦将对象置入容器内，便损失了该对象的型别信息。

（3）在各种Lists中，最好的做法是以ArrayList作为缺省选择。当插入、删除频繁时，使用LinkedList()； Vector总是比ArrayList慢，所以要尽量避免使用。

（4）在各种Sets中，HashSet通常优于HashTree（插入、查找）。只有当需要产生一个经过排序的序列，才用TreeSet。HashTree存在的唯一理由：能够维护其内元素的排序状态。

（5） 在各种Maps中，HashMap用于快速查找。

（6）当元素个数固定，用Array，因为Array效率是最高的。

# 4. BlockingQueue成员详细介绍

## 4.1 ArrayBlockingQueue
（1）基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据对象，这是一个常用的阻塞队列，除了一个定长数组外，ArrayBlockingQueue内部还保存着两个整形变量，分别标识着队列的头部和尾部在数组中的位置。

（2）ArrayBlockingQueue在生产者放入数据和消费者获取数据，都是共用同一个锁对象，由此也意味着两者无法真正并行运行，这点尤其不同于LinkedBlockingQueue。

（3）按照实现原理来分析，ArrayBlockingQueue完全可以采用分离锁，从而实现生产者和消费者操作的完全并行运行。Doug Lea之所以没这样去做，也许是因为ArrayBlockingQueue的数据写入和获取操作已经足够轻巧，以至于引入独立的锁机制，除了给代码带来额外的复杂性外，其在性能上完全占不到任何便宜。

（4）ArrayBlockingQueue和LinkedBlockingQueue间还有一个明显的不同之处在于，前者在插入或删除元素时不会产生或销毁任何额外的对象实例，而后者则会生成一个额外的Node对象。这在长时间内需要高效并发地处理大批量数据的系统中，其对于GC的影响还是存在一定的区别。而在创建ArrayBlockingQueue时，我们还可以控制对象的内部锁是否采用公平锁，默认采用非公平锁。

## 4.2 LinkedBlockingQueue
（1）基于链表的阻塞队列，同ArrayListBlockingQueue类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立即返回；只有当队列缓冲区达到最大值缓存容量时（LinkedBlockingQueue可以通过构造函数指定该值），才会阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者这端的处理也基于同样的原理。

（2）LinkedBlockingQueue之所以能够高效的处理并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。
（3）作为开发者，我们需要注意的是，如果构造一个LinkedBlockingQueue对象，而没有指定其容量大小，LinkedBlockingQueue会默认一个类似无限大小的容量（Integer.MAX_VALUE），这样的话，如果生产者的速度一旦大于消费者的速度，也许还没有等到队列满阻塞产生，系统内存就有可能已被消耗殆尽了。

### 4.2.1 ArrayBlockingQueue VS LinkedBlockingQueue
1. 队列中锁的实现不同
ArrayBlockingQueue实现的队列中的锁是没有分离的，即生产和消费用的是同一个锁；

LinkedBlockingQueue实现的队列中的锁是分离的，即生产用的是putLock，消费是takeLock

2. 在生产或消费时操作不同
ArrayBlockingQueue实现的队列中在生产和消费的时候，是直接将枚举对象插入或移除的；

LinkedBlockingQueue实现的队列中在生产和消费的时候，需要把枚举对象转换为Node<E>进行插入或移除，会影响性能

3. 队列大小初始化方式不同
ArrayBlockingQueue实现的队列中必须指定队列的大小；

LinkedBlockingQueue实现的队列中可以不指定队列的大小，但是默认是Integer.MAX_VALUE

## 4.3 DelayQueue
（1）DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。

（2）DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。

（3）使用场景：DelayQueue使用场景较少，但都相当巧妙，常见的例子比如使用一个DelayQueue来管理一个超时未响应的连接队列。

## 4.4 PriorityBlockingQueue
（1）基于优先级的阻塞队列（优先级的判断通过构造函数传入的Comparator对象来决定），但需要注意的是PriorityBlockingQueue并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。

（2）在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁。

## 4.5 SynchronousQueue
声明一个SynchronousQueue有两种不同的方式，它们之间有着不太一样的行为。公平模式和非公平模式的区别:如果采用公平模式：SynchronousQueue会采用公平锁，并配合一个FIFO队列来阻塞多余的生产者和消费者，从而体系整体的公平策略；但如果是非公平模式（SynchronousQueue默认）：SynchronousQueue采用非公平锁，同时配合一个LIFO队列来管理多余的生产者和消费者，而后一种模式，如果生产者和消费者的处理速度有差距，则很容易出现饥渴的情况，即可能有某些生产者或者是消费者的数据永远都得不到处理。


## 4.6 TransferQueue和其实现类LinkedTransferQueue。
（1）TransferQueue继承了BlockingQueue（BlockingQueue又继承了Queue）并扩展了一些新方法。BlockingQueue（和Queue）是Java 5中加入的接口，它是指这样的一个队列：当生产者向队列添加元素但队列已满时，生产者会被阻塞；当消费者从队列移除元素但队列为空时，消费者会被阻塞。

（2）TransferQueue则更进一步，生产者会一直阻塞直到所添加到队列的元素被某一个消费者所消费（不仅仅是添加到队列里就完事）。新添加的transfer方法用来实现这种约束。顾名思义，阻塞就是发生在元素从一个线程transfer到另一个线程的过程中，它有效地实现了元素在线程之间的传递（以建立Java内存模型中的happens-before关系的方式）。

（3）TransferQueue相比SynchronousQueue用处更广、更好用，因为你可以决定是使用BlockingQueue的方法（例如put方法）还是确保一次传递完成（即transfer方法）。在队列中已有元素的情况下，调用transfer方法，可以确保队列中被传递元素之前的所有元素都能被处理。

（4）Doug Lea说从功能角度来讲，LinkedTransferQueue实际上是ConcurrentLinkedQueue、SynchronousQueue（公平模式）和LinkedBlockingQueue的超集。而且LinkedTransferQueue更好用，因为它不仅仅综合了这几个类的功能，同时也提供了更高效的实现。

# 5. BlockingQueue定义的常用方法如下:

|操作		|抛出异常		|特殊值		|阻塞 	|超时		|
|-------|-----------|-----------|-------|-------|
|插入		|add(e)		|offer(e)	|put(e)	|offer(e, time, unit)|
|移除		|remove()	|poll()		|take()	|poll(time, unit)|
|检查		|element()	|peek()		|不可用	|不可用	|

（1）add方法和offer方法的区别在于超出容量限制时前者抛出异常，后者返回false；

（2）remove方法和poll方法都从队列中拿掉元素并返回，但是他们的区别在于空队列下操作前者抛出异常，而后者返回null；

（3）element方法和peek方法都返回队列顶端的元素，但是不把元素从队列中删掉，区别在于前者在空队列的时候抛出异常，后者返回null。


# 6. ConcurrentSkipListMap
（1）跳表是一种采用了用空间换时间思想的数据结构。它会随机地将一些节点提升到更高的层次，以创建一种逐层的数据结构，以提高操作的速度。

（2）在非多线程的情况下，应当尽量使用TreeMap。此外对于并发性相对较低的并行程序可以使用Collections.synchronizedSortedMap将TreeMap进行包装，也可以提供较好的效率。对于高并发程序，应当使用ConcurrentSkipListMap，能够提供更高的并发度。

（3）ConcurrentSkipListMap提供了一种线程安全的并发访问的排序映射表。内部是SkipList（跳表）结构实现，在理论上能够在O(log(n))时间内完成查找、插入、删除操作。注意，调用ConcurrentSkipListMap的size时，由于多个线程可以同时对映射表进行操作，所以映射表需要遍历整个链表才能返回元素个数，这个操作是个O(log(n))的操作。所以在多线程程序中，如果需要对Map的键值进行排序时，请尽量使用ConcurrentSkipListMap，可能得到更好的并发度。

## 6.1 Skiplist的性质
(1) 由很多层结构组成，level是通过一定的概率随机产生的。

(2) 每一层都是一个有序的链表，默认是升序，也可以根据创建映射时所提供的Comparator进行排序，具体取决于使用的构造方法。

(3) 最底层(Level 1)的链表包含所有元素。

(4) 如果一个元素出现在Level i 的链表中，则它在Level i 之下的链表也都会出现。

(5) 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素

# 7. CopyOnWriteArrayList
（1）CopyOnWriteArrayList是jdk concurrent包中提供的一个非阻塞型的，线程安全的List实现。

（2）CopyOnWriteArrayList在进行数据修改时，都不会对数据进行锁定，每次修改时，先拷贝整个数组，然后修改其中的一些元素，完成上述操作后，替换整个数组的指针。

（3）对CopyOnWriteArrayList进行读取时，也不进行数据锁定，直接返回需要查询的数据，如果需要返回整个数组，那么会将整个数组拷贝一份，再返回，保证内部array在任何情况下都是只读的。

（4）这里同样没有使用synchronized关键字，而是使用ReentrantLock。和ArrayList不同的是，这里每次都会创建一个新的object数组，大小比之前数组大1。将之前的数组复制到新数组，并将新加入的元素加到数组末尾。

（5）从上可见，CopyOnWriteArrayList基于ReentrantLock保证了增加元素和删除元素动作的互斥。在读操作上没有任何锁，这样就保证了读的性能，带来的副作用是有时候可能会读取到脏数据。

# 8. ConcurrentHashMap的锁分段技术

![ConcurrentHashMap](../photo/ConcurrentHashMap.jpg)

（1）HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

（2）ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护者一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。

（3）ConcurrentHashMap的get，JDK16的解释如下，JDK17换了实现方式：UNSAFE.getObjectVolatile
如果找到了所求的结点，判断它的值如果非空就直接返回，否则在有锁的状态下再读一次。这似乎有些费解，理论上结点的值不可能为空，这是因为 put的时候就进行了判断，如果为空就要抛NullPointerException。空值的唯一源头就是HashEntry中的默认值，因为 HashEntry中的value不是final的，非同步读取有可能读取到空值。仔细看下put操作的语句：tab[index] = new HashEntry<K,V>(key, hash, first, value)，在这条语句中，HashEntry构造函数中对value的赋值以及对tab[index]的赋值可能被重新排序，这就可能导致结点的值为空

# 9. 线程原理&使用：Runnable，Callable，Future，Thread，ThreadFactory，ThreadGroup，Executor，Executors，ThreadPoolExecutor，ForkJoinPool
## Callable
Callable是类似于Runnable的接口，实现Callable接口的类和实现Runnable的类都是可被其他线程执行的任务。

Callable的接口定义如下；
```java
public interface Callable<V> {

      V   call()   throws Exception;

}
```
## 9.1 Callable和Runnable的区别如下：
I	Callable定义的方法是call，而Runnable定义的方法是run。

II	Callable的call方法可以有返回值，而Runnable的run方法不能有返回值。

III	Callable的call方法可抛出异常，而Runnable的run方法不能抛出异常。  

##9.2  Future
Future表示异步计算的结果，它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。Future的cancel方法可以取消任务的执行，它有一布尔参数，参数为 true 表示立即中断任务的执行，参数为 false 表示允许正在运行的任务运行完成。Future的 get 方法等待计算完成，获取计算结果。

## 9.3 Thread、Runnable、Callable
其中Runnable实现的是void run()方法，Callable实现的是 V call()方法，并且可以返回执行结果，其中Runnable可以提交给Thread来包装下，直接启动一个线程来执行，而Callable则一般都是提交给ExecuteService来执行。

## 9.4 Executor
简单来说，Executor就是Runnable和Callable的调度容器，Future就是对于具体的调度任务的执行结果进行查看，最为关键的是Future可以检查对应的任务是否已经完成，也可以阻塞在get方法上一直等待任务返回结果。Runnable和Callable的差别就是Runnable是没有结果可以返回的，就算是通过Future也看不到任务调度的结果的。

## 9.5 FutureTask
FutureTask实现了两个接口，Runnable和Future，所以它既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值，那么这个组合的使用有什么好处呢？假设有一个很耗时的返回值需要计算，并且这个返回值不是立刻需要的话，那么就可以使用这个组合，用另一个线程去计算返回值，而当前线程在使用这个返回值之前可以做其它的操作，等到需要这个返回值时，再通过Future得到。

## 9.6 ThreadFactory
Java提供ThreadFactory接口，用来实现一个Thread对象工厂。Java并发API的一些高级工具，如执行者框架（Executor framework）或Fork/Join框架（Fork/Join framework），使用线程工厂创建线程。

ThreadFactory接口只有一个方法，newThread()方法接收一个Runnable对象作为参数，并且返回一个用来执行Runnable对象的Thread对象
```java
class SimpleThreadFactory implements ThreadFactory {  
  public Thread newThread(Runnable r) {  
    return new Thread(r);  
  }  
}  
```

## 9.7 ThreadGroup
当创建了好几个线程的时候，很多线程的工作任务是类似或者一致的，这样我们就可以使用ThreadGroup来管理他们，ThreadGroup可以随时的获取在他里面的线程的运行状态，信息，或者一条命令关闭掉这个group里面的所有线程，非常的简单实用。

## 9.8 ExecutorService
（1）Future.class，异步计算的结果对象，get方法会阻塞线程直至真正的结果返回。

（2）Callable.class，用于异步执行的可执行对象，call方法有返回值，它和Runnable接口很像，都提供了在其他线程中执行的方法，二者的区别在于：Runnable没有返回值，Callable有；Callable的call方法声明了异常抛出，而Runnable没有。

（3）RunnableFuture.class，实现自Runnable和Future的子接口，成功执行run方法可以完成它自身这个Future并允许访问其结果，它把任务执行和结果对象放到一起了。

（4）FutureTask.class，RunnableFuture的实现类，可取消的异步计算任务，仅在计算完成时才能获取结果，一旦计算完成，就不能再重新开始或取消计算；它的取消任务方法cancel(boolean mayInterruptIfRunning)接收一个boolean参数表示在取消的过程中是否需要设置中断。

（5）Executor.class，执行提交任务的对象，只有一个execute方法。

（6）Executors.class，辅助类和工厂类，帮助生成下面这些ExecutorService。

（7）ExecutorService.class，Executor的子接口，管理执行异步任务的执行器，AbstractExecutorService提供了默认实现。

## 9.9 Executor，Executors
执行者框架（Executor framework）是一种机制，允许你将线程的创建与执行分离。它是基于Executor和ExecutorService接口及其实现这两个接口的ThreadPoolExecutor类。它有一个内部的线程池和提供允许你提交两种任务给线程池执行的方法。这些任务是：Runnable接口，实现没有返回结果的任务；Callable接口，实现返回结果的任务。

（1）Single Thread Executor : 只有一个线程的线程池，因此所有提交的任务是顺序执行，代码： Executors.newSingleThreadExecutor()

（2）Cached Thread Pool : 线程池里有很多线程需要同时执行，老的可用线程将被新的任务触发重新执行，如果线程超过60秒内没执行，那么将被终止并从池中删除，代码：Executors.newCachedThreadPool()

（3）Fixed Thread Pool : 拥有固定线程数的线程池，如果没有任务执行，那么线程会一直等待，代码： Executors.newFixedThreadPool()

（4）Scheduled Thread Pool : 用来调度即将执行的任务的线程池，代码：Executors.newScheduledThreadPool()


## 9.10 ThreadPoolExecutor最常用构造方法为：
```java
ThreadPoolExecutor(int corePoolSize,   
                   int maximumPoolSize,   
                   long keepAliveTime,   
                   TimeUnit unit,   
                   BlockingQueue<Runnable> workQueue,   
                   RejectedExecutionHandler handler)
```
（1）当一个任务通过execute(Runnable)方法欲添加到线程池时：
+ 如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。
+ 如果此时线程池中的数量等于 corePoolSize，但是缓冲队列 workQueue未满，那么任务被放入缓冲队列。
+ 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maximumPoolSize，建新的线程来处理被添加的任务。
+ 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过 handler所指定的策略来处理此任务。也就是：处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、最大线程maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。
+ 当线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。

（2）核心和最大池大小
+ ThreadPoolExecutor 将根据 corePoolSize（参见 getCorePoolSize()）和 maximumPoolSize（参见 getMaximumPoolSize()）设置的边界自动调整池大小。+ + 当新任务在方法 execute(java.lang.Runnable) 中提交时，如果运行的线程少于 corePoolSize，则创建新线程来处理请求，即使其他辅助线程是空闲的。
+ 如果运行的线程多于 corePoolSize而少于 maximumPoolSize，则仅当队列满时才创建新线程。
+ 如果设置的 corePoolSize 和 maximumPoolSize 相同，则创建了固定大小的线程池。
+ 如果将 maximumPoolSize 设置为基本的无界值（如 Integer.MAX_VALUE），则允许池适应任意数量的并发任务。
+ 在大多数情况下，核心和最大池大小仅基于构造来设置，不过也可以使用 setCorePoolSize(int) 和 setMaximumPoolSize(int) 进行动态更改。

（3）排队及策略
所有 BlockingQueue 都可用于传输和保持提交的任务。可以使用此队列与池大小进行交互：

+ 如果运行的线程少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。
+ 如果运行的线程等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。
+ 如果无法将请求加入队列，则创建新的线程，除非创建此线程超出 maximumPoolSize，在这种情况下，任务将被拒绝。

排队有三种通用策略：
+ 直接提交。工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集合时出现锁定。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

+ 无界队列。使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙的情况下将新任务加入队列。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

+ 有界队列。当使用有限的 maximumPoolSizes 时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。

（4）拒绝任务的处理策略 （这个和参数handler设置相关）
当 Executor 已经关闭，并且 Executor 将有限边界用于最大线程和工作队列容量，且已经饱和时，在方法 execute(java.lang.Runnable) 中提交的新任务将被拒绝。在以上两种情况下，execute 方法都将调用其 RejectedExecutionHandler 的 RejectedExecutionHandler.rejectedExecution(java.lang.Runnable, java.util.concurrent.ThreadPoolExecutor) 方法。下面提供了四种预定义的处理程序策略：

+ 在默认的 ThreadPoolExecutor.AbortPolicy 中，处理程序遭到拒绝将抛出运行时 RejectedExecutionException。
+ 在 ThreadPoolExecutor.CallerRunsPolicy 中，线程调用运行该任务的 execute 本身。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。
+ 在 ThreadPoolExecutor.DiscardPolicy 中，不能执行的任务将被删除。
+ 在 ThreadPoolExecutor.DiscardOldestPolicy 中，如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）。
+ 定义和使用其他种类的 RejectedExecutionHandler 类也是可能的，但这样做需要非常小心，尤其是当策略仅用于特定容量或排队策略时


# 10. Fork-join框架
JDK7引入的并行框架，它把流程划分成fork（分解）+join（合并）两个步骤（怎么那么像MapReduce？），传统线程池来实现一个并行任务的时候，经常需要花费大量的时间去等待其他线程执行任务的完成，但是fork-join框架使用work stealing技术缓解了这个问题：

1. 每个工作线程都有一个双端队列，当分给每个任务一个线程去执行的时候，这个任务会放到这个队列的头部；
2. 当这个任务执行完毕，需要和另外一个任务的结果执行合并操作，可是那个任务却没有执行的时候，不会干等，而是把另一个任务放到队列的头部去，让它尽快执行；
3. 当工作线程的队列为空，它会尝试从其他线程的队列尾部偷一个任务过来；
4. 取得的任务可以被进一步分解。

# 11. 读写锁
```java
public class ReadWriteLock{
	private int readers = 0;
	private int writers = 0;
	private int writeRequests = 0;

	public synchronized void lockRead()
		throws InterruptedException{
		while(writers > 0 || writeRequests > 0){
			wait();
		}
		readers++;
	}

	public synchronized void unlockRead(){
		readers--;
		notifyAll();
	}

	public synchronized void lockWrite()
		throws InterruptedException{
		writeRequests++;

		while(readers > 0 || writers > 0){
			wait();
		}
		writeRequests--;
		writers++;
	}

	public synchronized void unlockWrite()
		throws InterruptedException{
		writers--;
		notifyAll();
	}
}
```
# 12. 可重入锁
（1）如果锁具备可重入性，则称作为可重入锁。

（2）像synchronized和ReentrantLock都是可重入锁，可重入性在我看来实际上表明了锁的分配机制。

（3）基于线程的分配，而不是基于方法调用的分配。

# 13. 可中断锁
（1）可中断锁：顾名思义，就是可以相应中断的锁。

（2）在Java中，synchronized就不是可中断锁，而Lock是可中断锁。

（3）如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

（4）在前面演示lockInterruptibly()的用法时已经体现了Lock的可中断性。

# 14. 公平锁
（1）公平锁即尽量以请求锁的顺序来获取锁。比如同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该所，这种就是公平锁。

（2）非公平锁即无法保证锁的获取是按照请求锁的顺序进行的。这样就可能导致某个或者一些线程永远获取不到锁。

（3）在Java中，synchronized就是非公平锁，它无法保证等待的线程获取锁的顺序。

（4）而对于ReentrantLock和ReentrantReadWriteLock，它默认情况下是非公平锁，但是可以设置为公平锁。

# 15. 读写锁
（1）读写锁将对一个资源（比如文件）的访问分成了2个锁，一个读锁和一个写锁。正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。

（2）ReadWriteLock就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。可以通过readLock()获取读锁，通过writeLock()获取写锁。


# 16. LockSupport与 Object Monitor的区别
LockSupport是针对特定线程来进行阻塞和解除阻塞操作的；而Object的wait()/notify()/notifyAll()是用来操作特定对象的等待集合的。

LockSupport与伪唤醒问题跟Object.wait()方法一样，park系列方法也会因为伪唤醒的原因返回。为了防止上述情况发生，park方法的使用需要遵守如下约定：
```java
while (!canProceed()) { ... LockSupport.park(this); }
```

# 17. Atomic原子类：CAS原理，基本类，引用类，数组类，属性的原子修改器
```java
public final boolean compareAndSet(int expect, int update) {   
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
 }
```
基本类：AtomicInteger、AtomicLong、AtomicBoolean；

引用类型：AtomicReference、AtomicStampedRerence、AtomicMarkableReference；

数组类型：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray

属性原子修改器（Updater）：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater

# 18. volatile 用法
在多线程并发编程中 synchronized 和 volatile 都扮演着重要的角色，volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。它在某些情况下比synchronized的开销更小。有volatile变量修饰的共享变量进行写操作的时候会多第二行汇编代码，通过查IA-32架构软件开发者手册可知，lock前缀的指令在多核处理器下会引发了两件事情。

（1）将当前处理器缓存行的数据会写回到系统内存。

（2）这个写回内存的操作会引起在其他CPU里缓存了该内存地址的数据无效。

# 19. synchronized 关键字
（1）修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；

（2）修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象；

（3）修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；

（4）修饰一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。

synchronized(Sync.class)实现了全局锁的效果。static synchronized方法，static方法可以直接类名加方法名调用，方法中无法使用this，所以它锁的不是this，而是类的Class对象，所以，static synchronized方法也相当于全局锁，相当于锁住了代码段。

A. 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类，该类所有的对象同一把锁。

B. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。

C. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。

# 20. ThreadLocal
ThreadLocal不是用来解决对象共享访问问题的，而主要是提供了保持对象的方法和避免参数传递的方便的对象访问方式。归纳了两点：

（1）每个线程中都有一个自己的ThreadLocalMap类对象，可以将线程自己的对象保持到其中，各管各的，线程可以正确的访问到自己的对象。

（2）将一个共用的ThreadLocal静态实例作为key，将不同对象的引用保存到不同线程的ThreadLocalMap中，然后在线程执行的各处通过这个静态ThreadLocal实例的get()方法取得自己线程保存的那个对象，避免了将这个对象作为参数传递的麻烦。

Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。

# 21. Java 四引用
Java 的引用主要分为强引用、软引用、弱引用、虚引用。

+ 强引用（StrongReference）：强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java 虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。

+ 软引用（SoftReference）：如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java 虚拟机就会把这个软引用加入到与之关联的引用队列中。

+ 弱引用（WeakReference）：弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。

+ 虚引用（PhantomReference）：“虚引用”顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

# 22. 线程 Heap 信息解析
```
"http-bio-8070-exec-7" daemon prio=10 tid=0x00007f9410004000 nid=0xc5c waiting on condition [0x00007f94439ee000]

线程名："http-bio-8070-exec-7"
线程优先级：prio=10
java线程ID：tid=0x00007f9410004000
native线程的id：nid=0xc5c
线程栈起始地址：[0x00007f94439ee000]
```

# 23. 日志的切分
java 自带日志系统，写数据为 write(int); 其实只写低8位，一字节

对文件的操作需要加锁，使用内部流 MeteredStream 来计数写的字节数。
```java
    // A metered stream is a subclass of OutputStream that
    //   (a) forwards all its output to a target stream
    //   (b) keeps track of how many bytes have been written
    //    通过 setOutputStream 来切换 writer
        // Rotate the set of output files
    private synchronized void rotate() {
        Level oldLevel = getLevel();
        setLevel(Level.OFF);

        super.close();
        for (int i = count-2; i >= 0; i--) {
            File f1 = files[i];
            File f2 = files[i+1];
            if (f1.exists()) {
                if (f2.exists()) {
                    f2.delete();
                }
                f1.renameTo(f2);
            }
        }
        try {
            open(files[0], false); //append=true，则在未满的文件中追加
        } catch (IOException ix) {
            // We don't want to throw an exception here, but we
            // report the exception to any registered ErrorManager.
            reportError(null, ix, ErrorManager.OPEN_FAILURE);

        }
        setLevel(oldLevel);
    }
```

每次输出日志时，通过 doLog -> public void log(LogRecord record) {
```java
for (Handler handler : logger.getHandlers()) {
    handler.publish(record);
}
```

调用 publish 方法，超过设置的 byte 数则 rotate(); 输出到另一个文件中！

```java
public synchronized void publish(LogRecord record) {
....
if (limit > 0 && meter.written >= limit) {
            // We performed access checks in the "init" method to make sure
            // we are only initialized from trusted code.  So we assume
            // it is OK to write the target files, even if we are
            // currently being called from untrusted code.
            // So it is safe to raise privilege here.
            AccessController.doPrivileged(new PrivilegedAction<Object>() {
                public Object run() {
                    rotate();
                    return null;
                }
            });
        }
```

# 24. java 序列化
@ see [对Java Serializable（序列化）的理解和总结](https://www.cnblogs.com/qq3111901846/p/7894532.html)

Java 序列化指的是将对象转换程字节格式并将对象状态保存在文件中，通常是`.ser`扩展名的文件。然后可以通过 `.ser` 文件重新创建 Java 对象，这个过程为反序列化

（1）Java中的Serializable接口和Externalizable接口有什么区别？

Externalizable 接口提供了两个方法 writeExternal() 和 readExternal()。这两个方法给我们提供了灵活处理 Java 序列化的方法，通过实现这个接口中的两个方法进行对象序列化可以替代 Java 中默认的序列化方法。正确的实现 Externalizable 接口可以大幅度的提高应用程序的性能。

（2）Serializable 接口中有几个方法？如果没有方法的话，那么这么设计 Serializable 接口的目的是什么？

Serializable 接口在 java.lang 包中，是 Java 序列化机制的核心组成部分。它里面没有包含任何方法，我们称这样的接口为标识接口。如果你的类实现了Serializable 接口，这意味着你的类被打上了“可以进行序列化”的标签，并且也给了编译器指示，可以使用序列化机制对这个对象进行序列化处理。

在 ObjectOutputStream 类中有如下代码：
```java
private void writeObject0(Object obj, boolean unshared) throws IOException {

    if (obj instanceof String) {
        writeString((String) obj, unshared);
    } else if (cl.isArray()) {
        writeArray(obj, desc, unshared);
    } else if (obj instanceof Enum) {
        writeEnum((Enum) obj, desc, unshared);
    } else if (obj instanceof Serializable) {
        writeOrdinaryObject(obj, desc, unshared);
    } else {
        if (extendedDebugInfo) {
            throw new NotSerializableException(cl.getName() + "\n"
                    + debugInfoStack.toString());
        } else {
            throw new NotSerializableException(cl.getName());
        }
    }

}
```
从上述代码可知，如果被写对象的类型是String，或数组，或Enum，或Serializable，那么就可以对该对象进行序列化，否则将抛出NotSerializableException。

（3）什么是 serialVersionUID ？如果你没有定义 serialVersionUID 意味着什么？

SerialVersionUID 应该是你的类中的一个 public static final 类型的常量，如果你的类中没有定义的话，那么编译器将抛出警告。如果你的类中没有制定 serialVersionUID ，那么 Java 编译器会根据类的成员变量和一定的算法生成用来表达对象的 serialVersionUID ，通常是用来表示类的哈希值（hash code）。结论是，如果你的类没有实现 SerialVersionUID ，那么如果你的类中如果加入或者改变成员变量，那么已经序列化的对象将无法反序列化。这是因为类的成员变量的改变意味这编译器生成的  SerialVersionUID 的值不同。Java 序列化过程是通过正确 SerialVersionUID 来对已经序列化的对象进行状态恢复。

（4）当对象进行序列化的时候，如果你不希望你的成员变量进行序列化，你怎么办？

如果你不希望你的对象中的成员变量的状态得以保存，你可以根据需求选择 transient 或者 static 类型的变量，这样的变量不参与 Java 序列化处理的过程。

（5）如果一个类中的成员变量是其它符合类型的 Java 类，而这个类没有实现 Serializable 接口，那么当对象序列化的时候会怎样？

如果你的一个对象进行序列化，而这个对象中包含另外一个引用类型的成员编程，而这个引用的类没有实现 Serializable 接口，那么当对象进行序列化的时候会抛出“NotSerializableException”的运行时异常。

（6）如果一个类是可序列化的，而他的超类没有，那么当进行反序列化的时候，那些从超类继承的实例变量的值是什么？

Java 中的序列化处理实例变量只会在所有实现了 Serializable 接口的继承支路上展开。所以当一个类进行反序列化处理的时候，超类没有实现 Serializable 接口，那么从超类继承的实例变量会通过为实现序列化接口的超类的构造函数进行初始化。

（7）你能够自定义序列化处理的代码吗？或者你能重载 Java 中默认的序列化方法吗？

答案是肯定的。我们都知道可以通过 ObjectOutputStream 中的 writeObject() 方法写入序列化对象，通过 ObjectInputStream 中的 readObject() 读入反序列化的对象。这些都是 Java 虚拟机提供给你的两个方法。如果你在你的类中定义了这两个方法，那么 JVM 就会用你的方法代替原有默认的序列化机制的方法。你可以通过这样的方式类自定义序列化和反序列化的行为。需要注意的一点是，最好将这两个方法定义为 private ，以防止他们被继承、重写和重载。也只有 JVM 可以访问到你的类中所有的私有方法，你不用担心方法私有不会被调用到，Java 序列化过程会正常工作。

（8）假设一个新的类的超类实现了 Serializable 接口，那么如何让这个新的子类不被序列化？

如果一个超类已经序列化了，那么无法通过是否实现什么接口的方式再避免序列化的过程了，但是也还有一种方式可以使用。那就是需要你在你的类中重新实现 writeObject() 和 readObject() 方法，并在方法实现中通过抛出 NotSerializableException 。

（9）在 Java 进行序列化和反序列化处理的时候，哪些方法被使用了？

Java 序列化是通过 java.io.ObjectOutputStream 这个类来完成的。这个类是一个过滤器流，这个类完成对底层字节流的包装来进行序列化处理。我们通过 ObjectOutputStream.writeObject(obj) 进行序列化，通过 ObjectInputStream.readObject() 进行反序列化。对 writeObject() 方法的调用会触发 Java 中的序列化机制。readObject() 方法用来将已经持久化的字节数据反向创建 Java 对象，该方法返回 Object 类型，需要强制转换成你需要的正确类型。

（10）假设你有一个类并且已经将这个类的某一个对象序列化存储了，那么如果你在这个类中加入了新的成员变量，那么在反序列化刚才那个已经存在的对象的时候会怎么样？

这个取决于这个类是否有 serialVersionUID 成员。通过上面的，我们已经知道如果你的类没有提供 serialVersionUID，那么编译器会自动生成，而这个 serialVersionUID 就是对象的 hash code 值。那么如果加入新的成员变量，重新生成的 serialVersionUID 将和之前的不同，那么在进行反序列化的时候就会产生 java.io.InvalidClassException 的异常。这就是为什么要建议为你的代码加入 serialVersionUID 的原因所在了。

（11）java反序列化时会将NULL值变成""字符!!
```java
	private void writeObject(ObjectOutputStream out) {
		try {
			PutField putFields = out.putFields();
			System.out.println("原密码:" + password);
			password = "encryption";//模拟加密
			putFields.put("password", password);
			System.out.println("加密后的密码" + password);
			out.writeFields();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	private void readObject(ObjectInputStream in) {
		try {
			GetField readFields = in.readFields();
			Object object = readFields.get("password", "");
			System.out.println("要解密的字符串:" + object.toString());
			password = "pass";//模拟解密,需要获得本地的密钥
		} catch (IOException e) {
			e.printStackTrace();
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}

	}
```


# 25. javabean
（1）javabean 是什么？

Bean的中文含义是“豆子”，顾名思义，JavaBean是指一段特殊的Java类，就是有默然构造方法，只有get，set的方法的java类的对象。

专业点解释是：JavaBean 定义了一组规则，JavaBean 就是遵循此规则的平常的Java对象。满足这三个条件:  
+ 实现 java.io.Serializable 接口

+ 提供无参数的构造器

+ 提供getter 和 setter方法访问它的属性.


简单地说，JavaBean 是用Java语言描述的软件组件模型，其实际上是一个类。这些类遵循一个接口格式，以便于使函数命名、底层行为以及继承或实现的行为，可以把类看作标准的JavaBean组件进行构造和应用。

JavaBean 一般分为可视化组件和非可视化组件两种。可视化组件可以是简单的GUI元素，如按钮或文本框，也可以是复杂的，如报表组件；非可视化组件没有GUI表现形式，用于封装业务逻辑、数据库操作等。其最大的优点在于可以实现代码的可重用性。JavaBean 又同时具有以下特性。

+ 易于维护、使用、编写。

+ 可实现代码的重用性。

+ 可移植性强，但仅限于Java工作平台。

+ 便于传输，不限于本地还是网络。

+ 可以以其他部件的模式进行工作。

对于有过其他语言编程经验的读者，可以将其看作类似微软的ActiveX的编程组件。但是区别在于 JavaBean 是跨平台的，而ActiveX组件则仅局限于Windows系统。总之，JavaBean 比较适合于那些需要跨平台的、并具有可视化操作和定制特性的软件组件。

JavaBean 组件与EJB（Enterprise JavaBean，企业级JavaBean）组件完全不同。EJB 是J2EE的核心，是一个用来创建分布式应用、服务器端以及基于Java应用的功能强大的组件模型。JavaBean 组件主要用于存储状态信息，而EJB组件可以存储业务逻辑。

（2）使用JavaBean的原因

程序中往往有重复使用的段落，JavaBean 就是为了能够重复使用而设计的程序段落，而且这些段落并不只服务于某一个程序，而且每个JavaBean都具有特定功能，当需要这个功能的时候就可以调用相应的JavaBean。从这个意义上来讲，JavaBean 大大简化了程序的设计过程，也方便了其他程序的重复使用。

JavaBean 传统应用于可视化领域，如AWT（窗口工具集）下的应用。而现在，JavaBean更多地应用于非可视化领域，同时，JavaBean 在服务器端的应用也表现出强大的优势。非可视化的JavaBean可以很好地实现业务逻辑、控制逻辑和显示页面的分离，现在多用于后台处理，使得系统具有更好的健壮性和灵活性。JSP + JavaBean 和JSP + JavaBean + Servlet 成为当前开发Web应用的主流模式。

（3）JavaBean的开发

在程序设计的过程中，JavaBean不是独立的。为了能够更好地封装事务逻辑、数据库操作而便于实现业务逻辑和前台程序的分离，操作的过程往往是先开发需要的JavaBean，再在适当的时候进行调用。但一个完整有效的JavaBean必然会包含一个属性，伴随若干个get/set（只读/只写）函数的变量来设计和运行的。JavaBean作为一个特殊的类，具有自己独有的特性。应该注意以下3个方面。

+ JavaBean 类必须有一个没有参数的构造函数。

+ JavaBean 类所有的属性最好定义为私有的。

+ JavaBean 类中定义函数setXxx() 和getXxx()来对属性进行操作。其中Xxx是首字母大写的私有变量名称

A JavaBean is just a standard

+ All properties private (use getters/setters)

+ A public no-argument constructor

+ Implements Serializable.

That's it. It's just a convention.
