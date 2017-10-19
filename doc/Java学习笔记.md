# 基本接口：List，Map，Set，Vector，Queue，Deque 
# 常用数据结构：ArrayList，HashMap，LinkedHashMap，HashSet，TreeMap
1. Collection 是对象集合， Collection 有两个子接口 List 和 Set
2. set map list的区别
``` 
	set －－其中的值不允许重复，无序的数据结构 
	list   －－其中的值允许重复，因为其为有序的数据结构 
	map－－成对的数据结构，健值必须具有唯一性（键不能同，否则值替换）
``` 
3. List 按对象进入的顺序保存对象，不做排序或编辑操作。
4. Set对每个对象只接受一次，并使用自己内部的排序方法（通常，你只关心某个元素是否属于Set,而不关心它的顺序--否则应该使用List）。
5. Map同样对每个元素保存一份，但这是基于"键"的，Map也有内置的排序，因而不关心元素添加的顺序。如果添加元素的顺序对你很重要，应该使用 LinkedHashSet或者LinkedHashMap.
6. List 可以通过下标 (1,2..)来取得值，值可以重复。Set 只能通过游标来取值，并且值是不能重复的
7. ArrayList ， Vector ， LinkedList 是 List 的实现类
```
	ArrayList 是线程不安全的， Vector 是线程安全的，这两个类底层都是由数组实现的
	LinkedList 是线程不安全的，底层是由链表实现的  
```
8. Map 是键值对集合
```
	HashTable 和 HashMap 是 Map 的实现类 
	HashTable 是线程安全的，不能存储 null 值
	HashMap 不是线程安全的，可以存储 null 值
```

## Collections类和Collection接口

1. Collections是针对集合类的一个帮助类，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。
2. Collection是最基本的集合接口，一个Collection代表一组Object，即Collection的元素（Elements）。一些 Collection允许相同的元素而另一些不行。一些能排序而另一些不行。Java SDK不提供直接继承自Collection的 类，Java SDK提供的类都是继承自Collection的“子接口”如List和Set。
3. 所有实现 Collection 接口的类都必须提供两个标准的构造函数：无参数的构造函数用于创建一个空的 Collection ，有一个 Collection 参数的构造函数用于创建一个新的 Collection ，这个新的 Collection与传入的 Collection 有相同的元素。后一个构造函数允许用户复制一个 Collection 。
4. 集合类的遍历:遍历通用Collection:如何遍历 Collection 中的每一个元素？不论 Collection 的实际类型如何，它都支持一个 iterator() 的方法，该方法返回一个迭代子，使用该迭代子即可逐一访问 Collection 中每一个元素。典型的用法如下：
```
Iterator it = collection.iterator(); // 获得一个迭代子  
　　while(it.hasNext()) {  
　　　Object obj = it.next(); // 得到下一个元素  
}  
```

## List接口，有序可重复的集合
	实际上有两种List: 一种是基本的ArrayList,其优点在于随机访问元素，另一种是更强大的LinkedList,它并不是为快速随机访问设计的，而是具有一套更通用的方法。 List : 次序是List最重要的特点：它保证维护元素特定的顺序。List为Collection添加了许多方法，使得能够向List中间插入与移除元素（这只推荐LinkedList使用。）一个List可以生成ListIterator,使用它可以从两个方向遍历List,也可以从List中间插入和移除元素。 

1. ArrayList类
	1) ArrayList实现了可变大小的数组。它允许所有元素，包括null。ArrayList没有同步。
	2) size，isEmpty，get，set方法运行时间为常数。但是add方法开销为分摊的常数，添加n个元素需要O(n)的时间。其他的方法运行时间为线性。
	3) 每个ArrayList实例都有一个容量（Capacity），即用于存储元素的数组的大小。这个容量可随着不断添加新元素而自动增加，但是增长算法 并没有定义。当需要插入大量元素时，在插入前可以调用ensureCapacity方法来增加ArrayList的容量以提高插入效率。
	4) 和LinkedList一样，ArrayList也是非同步的（unsynchronized）。
	5) 由数组实现的List。允许对元素进行快速随机访问，但是向List中间插入与移除元素的速度很慢。ListIterator只应该用来由后向前遍历ArrayList,而不是用来插入和移除元素。因为那比LinkedList开销要大很多。
2. Vector类
	Vector非常类似ArrayList，但是Vector是同步的。由Vector创建的Iterator，虽然和ArrayList创建的Iterator是同一接口，但是，因为Vector是同步的，当一个Iterator被创建而且正在被使用，另一个线程改变了Vector的状态（例如，添加或删除了一些元素），这时调用Iterator的方法时将抛出ConcurrentModificationException，因此必须捕获该异常。
3. LinkedList类
	LinkedList实现了List接口，允许null元素。此外LinkedList提供额外的get，remove，insert方法在 LinkedList的首部或尾部。如下列方法：addFirst(), addLast(), getFirst(), getLast(), removeFirst() 和 removeLast(), 这些方法 （没有在任何接口或基类中定义过）。这些操作使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。
	注意LinkedList没有同步方法。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List：
	List list = Collections.synchronizedList(new LinkedList(...));
4. Stack 类
	Stack继承自Vector，实现一个后进先出的堆栈。Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop方法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置。Stack刚创建后是空栈。

## Set接口，代表无序，不可重复的集合

	Set具有与Collection完全一样的接口，因此没有任何额外的功能，不像前面有两个不同的List。实际上Set就是Collection,只是行为不同。（这是继承与多态思想的典型应用：表现不同的行为。）Set不保存重复的元素（至于如何判断元素相同则较为负责） 
	Set : 存入Set的每个元素都必须是唯一的，因为Set不保存重复元素。加入Set的元素必须定义equals()方法以确保对象的唯一性。Set与Collection有完全一样的接口。Set接口不保证维护元素的次序。 
1. HashSet 
     为快速查找设计的Set。存入HashSet的对象必须定义hashCode()。 
2. TreeSet 
     保存次序的Set, 底层为树结构。使用它可以从Set中提取有序的序列。 
3. LinkedHashSet 
     具有HashSet的查询速度，且内部使用链表维护元素的顺序（插入的次序）。于是在使用迭代器遍历Set时，结果会按元素插入的次序显示	

## Map接口：映射

	Map没有继承Collection接口， Map 提供 key 到 value 的映射，你可以通过“键”查找“值”。一个 Map 中不能包含相同的 key ，每个 key 只能映射一个 value 。 Map 接口提供3种集合的视图， Map 的内容可以被当作一组 key 集合，一组 value 集合，或者一组 key-value 映射。方法 put(Object key, Object value) 添加一个“值” ( 想要得东西 ) 和与“值”相关联的“键” (key) ( 使用它来查找 ) 。方法get(Object key) 返回与给定“键”相关联的“值”。可以用 containsKey() 和 containsValue() 测试 Map 中是否包含某个“键”或“值”。 标准的 Java 类库中包含了几种不同的 Map ： HashMap, TreeMap, LinkedHashMap, WeakHashMap, IdentityHashMap 。它们都有同样的基本接口 Map ，但是行为、效率、排序策略、保存对象的生命周期和判定“键”等价的策略等各不相同。
	Map 同样对每个元素保存一份，但这是基于 " 键"的， Map 也有内置的排序，因而不关心元素添加的顺序。如果添加元素的顺序对你很重要，应该使用 LinkedHashSet 或者 LinkedHashMap.

## 1. Hashtable类
	Hashtable继承Map接口，实现一个key-value映射的哈希表。任何非空（non-null）的对象都可作为key或者value。Hashtable是同步的。添加数据使用 put(key, value) ，取出数据使用get(key)，这两个基本操作的时间开销为常数。Hashtable 通过初始化容量 (initial capacity) 和负载因子 (load factor) 两个参数调整性能。通常缺省的 load factor0.75 较好地实现了时间和空间的均衡。增大 load factor 可以节省空间但相应的查找时间将增大，这会影响像get和 put 这样的操作。

	由于作为 key 的对象将通过计算其散列函数来确定与之对应的 value 的位置，因此任何作为 key 的对象都必须实现 hashCode 方法和 equals 方法。 hashCode 方法和 equals 方法继承自根类 Object ，如果你用自定义的类当作 key 的话，要相当小心，按照散列函数的定义，如果两个对象相同，即 obj1.equals(obj2)=true，则它们的 hashCode 必须相同，但如果两个对象不同，则它们的 hashCode 不一定不同，如果两个不同对象的 hashCode 相同，这种现象称为冲突，冲突会导致操作哈希表的时间开销增大，所以尽量定义好的 hashCode() 方法，能加快哈希表的操作。

	如果相同的对象有不同的 hashCode ，对哈希表的操作会出现意想不到的结果（期待的 get 方法返回null
 ），要避免这种问题，只需要牢记一条：要同时复写 equals 方法和 hashCode 方法，而不要只写其中一个。Hashtable 是同步的。 

## 2. HashMap类
　　HashMap和Hashtable类似，也是基于hash散列表的实现。不同之处在于 HashMap是非同步的，并且允许null，即null value和null key。但是将HashMap视为Collection时 （values()方法可返回Collection），其迭代子操作时间开销和HashMap的容量成比例。因此，如果迭代操作的性能相当重要的话，不要 将HashMap的初始化容量设得过高，或者load factor过低。
　　 LinkedHashMap 类：类似于 HashMap ，但是迭代遍历它时，取得“键值对”的顺序是其插入次序，或者是最近最少使用 (LRU) 的次序。只比 HashMap 慢一点。而在迭代访问时发而更快，因为它使用链表维护内部次序。

## 3. WeakHashMap类 （弱键（ weak key ））
　　WeakHashMap是一种改进的HashMap，它是为解决特殊问题设计的，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收。

## 4. TreeMap 类
	基于红黑树数据结构的实现。查看“键”或“键值对”时，它们会被排序 ( 次序由 Comparabel 或 Comparator 决定 ) 。 TreeMap 的特点在于，你得到的结果是经过排序的。 TreeMap 是唯一的带有 subMap() 方法的 Map ，它可以返回一个子树。

## 5. IdentifyHashMap 类
	使用 == 代替 equals() 对“键”作比较的 hash map 。专为解决特殊问题而设计。

# 如何选择

1. 容器类和Array的区别、择取
      1)容器类仅能持有对象引用（指向对象的指针），而不是将对象信息copy一份至数列某位置。
      2)一旦将对象置入容器内，便损失了该对象的型别信息。
2、
     1)  在各种Lists中，最好的做法是以ArrayList作为缺省选择。当插入、删除频繁时，使用LinkedList()；
           Vector总是比ArrayList慢，所以要尽量避免使用。
      2) 在各种Sets中，HashSet通常优于HashTree（插入、查找）。只有当需要产生一个经过排序的序列，才用TreeSet。
           HashTree存在的唯一理由：能够维护其内元素的排序状态。
      3) 在各种Maps中，HashMap用于快速查找。
      4)  当元素个数固定，用Array，因为Array效率是最高的。

# 注意：
1、Collection没有get()方法来取得某个元素。只能通过iterator()遍历元素。
2、Set和Collection拥有一模一样的接口。
3、List，可以通过get()方法来一次取出一个元素。使用数字来选择一堆对象中的一个，get(0)...。(add/get)
4、一般使用ArrayList。用LinkedList构造堆栈stack、队列queue。
5、Map用 put(k,v) / get(k)，还可以使用containsKey()/containsValue()来检查其中是否含有某个key/value。HashMap会利用对象的hashCode来快速找到key。
6、Map中元素，可以将key序列、value序列单独抽取出来。使用keySet()抽取key序列，将map中的所有keys生成一个Set。使用values()抽取value序列，将map中的所有values生成一个Collection。为什么一个生成Set，一个生成Collection？那是因为，key总是独一无二的，value允许重复。

Concurrent：BlockingQueue，TransferQueue，ConcurrentHashMap，CopyOnWriteArrayList，ConcurrentSkipListMap，
=========================================================================================================
BlockingQueue成员详细介绍
=========================
1. ArrayBlockingQueue
      基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据对象，这是一个常用的阻塞队列，除了一个定长数组外，ArrayBlockingQueue内部还保存着两个整形变量，分别标识着队列的头部和尾部在数组中的位置。ArrayBlockingQueue在生产者放入数据和消费者获取数据，都是共用同一个锁对象，由此也意味着两者无法真正并行运行，这点尤其不同于LinkedBlockingQueue；按照实现原理来分析，ArrayBlockingQueue完全可以采用分离锁，从而实现生产者和消费者操作的完全并行运行。Doug Lea之所以没这样去做，也许是因为ArrayBlockingQueue的数据写入和获取操作已经足够轻巧，以至于引入独立的锁机制，除了给代码带来额外的复杂性外，其在性能上完全占不到任何便宜。 ArrayBlockingQueue和LinkedBlockingQueue间还有一个明显的不同之处在于，前者在插入或删除元素时不会产生或销毁任何额外的对象实例，而后者则会生成一个额外的Node对象。这在长时间内需要高效并发地处理大批量数据的系统中，其对于GC的影响还是存在一定的区别。而在创建ArrayBlockingQueue时，我们还可以控制对象的内部锁是否采用公平锁，默认采用非公平锁。

2. LinkedBlockingQueue
      基于链表的阻塞队列，同ArrayListBlockingQueue类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立即返回；只有当队列缓冲区达到最大值缓存容量时（LinkedBlockingQueue可以通过构造函数指定该值），才会阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者这端的处理也基于同样的原理。而LinkedBlockingQueue之所以能够高效的处理并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。
      作为开发者，我们需要注意的是，如果构造一个LinkedBlockingQueue对象，而没有指定其容量大小，LinkedBlockingQueue会默认一个类似无限大小的容量（Integer.MAX_VALUE），这样的话，如果生产者的速度一旦大于消费者的速度，也许还没有等到队列满阻塞产生，系统内存就有可能已被消耗殆尽了。

3. DelayQueue
      DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。使用场景：DelayQueue使用场景较少，但都相当巧妙，常见的例子比如使用一个DelayQueue来管理一个超时未响应的连接队列。

4. PriorityBlockingQueue
      基于优先级的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定），但需要注意的是PriorityBlockingQueue并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁。

5. SynchronousQueue
　　声明一个SynchronousQueue有两种不同的方式，它们之间有着不太一样的行为。公平模式和非公平模式的区别:如果采用公平模式：SynchronousQueue会采用公平锁，并配合一个FIFO队列来阻塞多余的生产者和消费者，从而体系整体的公平策略；但如果是非公平模式（SynchronousQueue默认）：SynchronousQueue采用非公平锁，同时配合一个LIFO队列来管理多余的生产者和消费者，而后一种模式，如果生产者和消费者的处理速度有差距，则很容易出现饥渴的情况，即可能有某些生产者或者是消费者的数据永远都得不到处理。

其中的一项是增加了接口TransferQueue和其实现类LinkedTransferQueue。
=================================================================
	TransferQueue继承了BlockingQueue（BlockingQueue又继承了Queue）并扩展了一些新方法。BlockingQueue（和Queue）是Java 5中加入的接口，它是指这样的一个队列：当生产者向队列添加元素但队列已满时，生产者会被阻塞；当消费者从队列移除元素但队列为空时，消费者会被阻塞。
	
	TransferQueue则更进一步，生产者会一直阻塞直到所添加到队列的元素被某一个消费者所消费（不仅仅是添加到队列里就完事）。新添加的transfer方法用来实现这种约束。顾名思义，阻塞就是发生在元素从一个线程transfer到另一个线程的过程中，它有效地实现了元素在线程之间的传递（以建立Java内存模型中的happens-before关系的方式）。
	
	当我第一次看到TransferQueue时，首先想到了已有的实现类SynchronousQueue。SynchronousQueue的队列长度为0，最初我认为这好像没多大用处，但后来我发现它是整个Java Collection Framework中最有用的队列实现类之一，特别是对于两个线程之间传递元素这种用例。

	TransferQueue相比SynchronousQueue用处更广、更好用，因为你可以决定是使用BlockingQueue的方法（译者注：例如put方法）还是确保一次传递完成（译者注：即transfer方法）。在队列中已有元素的情况下，调用transfer方法，可以确保队列中被传递元素之前的所有元素都能被处理。Doug Lea说从功能角度来讲，LinkedTransferQueue实际上是ConcurrentLinkedQueue、SynchronousQueue（公平模式）和LinkedBlockingQueue的超集。而且LinkedTransferQueue更好用，因为它不仅仅综合了这几个类的功能，同时也提供了更高效的实现。

BlockingQueue定义的常用方法如下:
====================================
 	抛出异常	特殊值	阻塞	超时
插入	add(e)	offer(e)	put(e)	offer(e, time, unit)
移除	remove()	poll()	take()	poll(time, unit)
检查	element()	peek()	不可用	不可用

1.    队列中锁的实现不同
       ArrayBlockingQueue实现的队列中的锁是没有分离的，即生产和消费用的是同一个锁；
       LinkedBlockingQueue实现的队列中的锁是分离的，即生产用的是putLock，消费是takeLock
    
2.    在生产或消费时操作不同
       ArrayBlockingQueue实现的队列中在生产和消费的时候，是直接将枚举对象插入或移除的；
       LinkedBlockingQueue实现的队列中在生产和消费的时候，需要把枚举对象转换为Node<E>进行插入或移除，会影响性能

3.    队列大小初始化方式不同
       ArrayBlockingQueue实现的队列中必须指定队列的大小；
       LinkedBlockingQueue实现的队列中可以不指定队列的大小，但是默认是Integer.MAX_VALUE

add方法和offer方法的区别在于超出容量限制时前者抛出异常，后者返回false；
remove方法和poll方法都从队列中拿掉元素并返回，但是他们的区别在于空队列下操作前者抛出异常，而后者返回null；
element方法和peek方法都返回队列顶端的元素，但是不把元素从队列中删掉，区别在于前者在空队列的时候抛出异常，后者返回null。

ConcurrentSkipListMap
=====================
	跳表是一种采用了用空间换时间思想的数据结构。它会随机地将一些节点提升到更高的层次，以创建一种逐层的数据结构，以提高操作的速度
	
	在非多线程的情况下，应当尽量使用TreeMap。此外对于并发性相对较低的并行程序可以使用Collections.synchronizedSortedMap将TreeMap进行包装，也可以提供较好的效率。对于高并发程序，应当使用ConcurrentSkipListMap，能够提供更高的并发度。

	ConcurrentSkipListMap提供了一种线程安全的并发访问的排序映射表。内部是SkipList（跳表）结构实现，在理论上能够在O(log(n))时间内完成查找、插入、删除操作。注意，调用ConcurrentSkipListMap的size时，由于多个线程可以同时对映射表进行操作，所以映射表需要遍历整个链表才能返回元素个数，这个操作是个O(log(n))的操作

	所以在多线程程序中，如果需要对Map的键值进行排序时，请尽量使用ConcurrentSkipListMap，可能得到更好的并发度

Skip list的性质
	(1) 由很多层结构组成，level是通过一定的概率随机产生的。
	(2) 每一层都是一个有序的链表，默认是升序，也可以根据创建映射时所提供的Comparator进行排序，具体取决于使用的构造方法。
	(3) 最底层(Level 1)的链表包含所有元素。
	(4) 如果一个元素出现在Level i 的链表中，则它在Level i 之下的链表也都会出现。
	(5) 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素

CopyOnWriteArrayList
=====================
	CopyOnWriteArrayList是jdk concurrent包中提供的一个非阻塞型的，线程安全的List实现。CopyOnWriteArrayList在进行数据修改时，都不会对数据进行锁定，每次修改时，先拷贝整个数组，然后修改其中的一些元素，完成上述操作后，替换整个数组的指针。对CopyOnWriteArrayList进行读取时，也不进行数据锁定，直接返回需要查询的数据，如果需要返回整个数组，那么会将整个数组拷贝一份，再返回，保证内部array在任何情况下都是只读的。

	这里同样没有使用synchronized关键字，而是使用ReentrantLock。和ArrayList不同的是，这里每次都会创建一个新的object数组，大小比之前数组大1。将之前的数组复制到新数组，并将新加入的元素加到数组末尾。

	从上可见，CopyOnWriteArrayList基于ReentrantLock保证了增加元素和删除元素动作的互斥。在读操作上没有任何锁，这样就保证了读的性能，带来的副作用是有时候可能会读取到脏数据

ConcurrentHashMap的锁分段技术
=============================
 	HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问

	ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护者一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁

线程原理&使用：
Runnable，Callable，Future，Thread，ThreadFactory，ThreadGroup
Executor，Executors，ThreadPoolExecutor，ForkJoinPool
=================================================================

    Callable与 Future 两功能是Java在后续版本中为了适应多并法才加入的，Callable是类似于Runnable的接口，实现Callable接口的类和实现Runnable的类都是可被其他线程执行的任务。

Callable的接口定义如下；

public interface Callable<V> { 

      V   call()   throws Exception; 

} 

Callable和Runnable的区别如下：

I    Callable定义的方法是call，而Runnable定义的方法是run。

II   Callable的call方法可以有返回值，而Runnable的run方法不能有返回值。

III  Callable的call方法可抛出异常，而Runnable的run方法不能抛出异常。  

Future表示异步计算的结果，它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。Future的cancel方法可以取消任务的执行，它有一布尔参数，参数为 true 表示立即中断任务的执行，参数为 false 表示允许正在运行的任务运行完成。Future的 get 方法等待计算完成，获取计算结果

Thread、Runnable、Callable，其中Runnable实现的是void run()方法，Callable实现的是 V call()方法，并且可以返回执行结果，其中Runnable可以提交给Thread来包装下，直接启动一个线程来执行，而Callable则一般都是提交给ExecuteService来执行。 

简单来说，Executor就是Runnable和Callable的调度容器，Future就是对于具体的调度任务的执行结果进行查看，最为关键的是Future可以检查对应的任务是否已经完成，也可以阻塞在get方法上一直等待任务返回结果。Runnable和Callable的差别就是Runnable是没有结果可以返回的，就算是通过Future也看不到任务调度的结果的。 

FutureTask实现了两个接口，Runnable和Future，所以它既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值，那么这个组合的使用有什么好处呢？假设有一个很耗时的返回值需要计算，并且这个返回值不是立刻需要的话，那么就可以使用这个组合，用另一个线程去计算返回值，而当前线程在使用这个返回值之前可以做其它的操作，等到需要这个返回值时，再通过Future得到

ThreadFactory
=============
Java提供ThreadFactory接口，用来实现一个Thread对象工厂。Java并发API的一些高级工具，如执行者框架（Executor framework）或Fork/Join框架（Fork/Join framework），使用线程工厂创建线程
ThreadFactory接口只有一个方法，newThread()方法接收一个Runnable对象作为参数，并且返回一个用来执行Runnable对象的Thread对象

class SimpleThreadFactory implements ThreadFactory {  
  public Thread newThread(Runnable r) {  
    return new Thread(r);  
  }  
}  

ThreadGroup
===========
当创建了好几个线程的时候，很多线程的工作任务是类似或者一致的，这样我们就可以使用ThreadGroup来管理他
们，ThreadGroup可以随时的获取在他里面的线程的运行状态，信息，或者一条命令关闭掉这个group里面的所有线
程，非常的简单实用

ExecutorService：
===================
Future.class，异步计算的结果对象，get方法会阻塞线程直至真正的结果返回
Callable.class，用于异步执行的可执行对象，call方法有返回值，它和Runnable接口很像，都提供了在其他线程中执行的方法，二者的区别在于：
Runnable没有返回值，Callable有
Callable的call方法声明了异常抛出，而Runnable没有
RunnableFuture.class，实现自Runnable和Future的子接口，成功执行run方法可以完成它自身这个Future并允许访问其结果，它把任务执行和结果对象放到一起了
FutureTask.class，RunnableFuture的实现类，可取消的异步计算任务，仅在计算完成时才能获取结果，一旦计算完成，就不能再重新开始或取消计算；它的取消任务方法cancel(boolean mayInterruptIfRunning)接收一个boolean参数表示在取消的过程中是否需要设置中断
Executor.class，执行提交任务的对象，只有一个execute方法
Executors.class，辅助类和工厂类，帮助生成下面这些ExecutorService
ExecutorService.class，Executor的子接口，管理执行异步任务的执行器，AbstractExecutorService提供了默认实现

Executor，Executors
===================
执行者框架（Executor framework）是一种机制，允许你将线程的创建与执行分离。它是基于Executor和ExecutorService接口及其实现这两个接口的ThreadPoolExecutor类。它有一个内部的线程池和提供允许你提交两种任务给线程池执行的方法。这些任务是：
	Runnable接口，实现没有返回结果的任务
	Callable接口，实现返回结果的任务

在使用线程池之前，必须知道如何去创建一个线程池，在Java5中，需要了解的是java.util.concurrent.Executors类的API，这个类提供大量创建连接池的静态方法，是必须掌握的。
一个线程池使用类 ExecutorService 的实例来表示，通过 ExecutorService 你可以提交任务，并进行调度执行。下面列举一些你可以通过 Executors 类来创建的线程池的类型：

	Single Thread Executor : 只有一个线程的线程池，因此所有提交的任务是顺序执行，代码： Executors.newSingleThreadExecutor()
Cached Thread Pool : 线程池里有很多线程需要同时执行，老的可用线程将被新的任务触发重新执行，如果线程超过60秒内没执行，那么将被终止并从池中删除，代码：Executors.newCachedThreadPool()
	Fixed Thread Pool : 拥有固定线程数的线程池，如果没有任务执行，那么线程会一直等待，代码： Executors.newFixedThreadPool()
Scheduled Thread Pool : 用来调度即将执行的任务的线程池，代码：Executors.newScheduledThreadPool()
	Single Thread Scheduled Pool : 只有一个线程，用来调度执行将来的任务，代码：Executors.newSingleThreadScheduledExecutor()



ThreadPoolExecutor最常用构造方法为：
================================
ThreadPoolExecutor(int corePoolSize,   
                   int maximumPoolSize,   
                   long keepAliveTime,   
                   TimeUnit unit,   
                   BlockingQueue<Runnable> workQueue,   
                   RejectedExecutionHandler handler) 
[ 1 ]当一个任务通过execute(Runnable)方法欲添加到线程池时：
	•如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。
	•如果此时线程池中的数量等于 corePoolSize，但是缓冲队列 workQueue未满，那么任务被放入缓冲队列。
	•如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maximumPoolSize，建新的线程来处理被添加的任务。
	•如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过 handler所指定的策略来处理此任务。也就是：处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、最大线程maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。
	•当线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。
[ 2 ]、核心和最大池大小
	ThreadPoolExecutor 将根据 corePoolSize（参见 getCorePoolSize()）和 maximumPoolSize（参见 getMaximumPoolSize()）设置的边界自动调整池大小。当新任务在方法 execute(java.lang.Runnable) 中提交时，如果运行的线程少于 corePoolSize，则创建新线程来处理请求，即使其他辅助线程是空闲的。如果运行的线程多于 corePoolSize 而少于 maximumPoolSize，则仅当队列满时才创建新线程。如果设置的 corePoolSize 和 maximumPoolSize 相同，则创建了固定大小的线程池。如果将 maximumPoolSize 设置为基本的无界值（如 Integer.MAX_VALUE），则允许池适应任意数量的并发任务。在大多数情况下，核心和最大池大小仅基于构造来设置，不过也可以使用 setCorePoolSize(int) 和 setMaximumPoolSize(int) 进行动态更改。
[ 3 ]、排队及策略
	所有 BlockingQueue 都可用于传输和保持提交的任务。可以使用此队列与池大小进行交互：
	•如果运行的线程少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。
	•如果运行的线程等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。
	•如果无法将请求加入队列，则创建新的线程，除非创建此线程超出 maximumPoolSize，在这种情况下，任务将被拒绝。
	排队有三种通用策略：
	•直接提交。工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集合时出现锁定。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。
	•无界队列。使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙的情况下将新任务加入队列。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。
	•有界队列。当使用有限的 maximumPoolSizes 时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。
[ 4 ]、拒绝任务的处理策略 （这个和参数handler设置相关）
	当 Executor 已经关闭，并且 Executor 将有限边界用于最大线程和工作队列容量，且已经饱和时，在方法 execute(java.lang.Runnable) 中提交的新任务将被拒绝。在以上两种情况下，execute 方法都将调用其 RejectedExecutionHandler 的 RejectedExecutionHandler.rejectedExecution(java.lang.Runnable, java.util.concurrent.ThreadPoolExecutor) 方法。下面提供了四种预定义的处理程序策略：
	•在默认的 ThreadPoolExecutor.AbortPolicy 中，处理程序遭到拒绝将抛出运行时 RejectedExecutionException。
	•在 ThreadPoolExecutor.CallerRunsPolicy 中，线程调用运行该任务的 execute 本身。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。
	•在 ThreadPoolExecutor.DiscardPolicy 中，不能执行的任务将被删除。
	•在 ThreadPoolExecutor.DiscardOldestPolicy 中，如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）。
	•定义和使用其他种类的 RejectedExecutionHandler 类也是可能的，但这样做需要非常小心，尤其是当策略仅用于特定容量或排队策略时


要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在Executors类里面提供了一些静态工厂，生成一些常用的线程池。
1. newSingleThreadExecutor
创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。
2.newFixedThreadPool
创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。
3. newCachedThreadPool
创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，
那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。
4.newScheduledThreadPool
创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。


Fork-join框架
===============
这是一个JDK7引入的并行框架，它把流程划分成fork（分解）+join（合并）两个步骤（怎么那么像MapReduce？），传统线程池来实现一个并行任务的时候，经常需要花费大量的时间去等待其他线程执行任务的完成，但是fork-join框架使用work stealing技术缓解了这个问题：
	1,	每个工作线程都有一个双端队列，当分给每个任务一个线程去执行的时候，这个任务会放到这个队列的头部；
	2,	当这个任务执行完毕，需要和另外一个任务的结果执行合并操作，可是那个任务却没有执行的时候，不会干等，而是把另一个任务放到队列的头部去，让它尽快执行；
	3,	当工作线程的队列为空，它会尝试从其他线程的队列尾部偷一个任务过来；
	4,	取得的任务可以被进一步分解。

线程同步
Synchronized
=============

Lock（可重入锁，读写锁，公平锁，可中断锁），LockSupport用法与Object.wait区别

锁
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

1.可重入锁

如果锁具备可重入性，则称作为可重入锁。

像synchronized和ReentrantLock都是可重入锁，可重入性在我看来实际上表明了锁的分配机制：

基于线程的分配，而不是基于方法调用的分配。

2.可中断锁

　　可中断锁：顾名思义，就是可以相应中断的锁。

　　在Java中，synchronized就不是可中断锁，而Lock是可中断锁。

　　如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

　　在前面演示lockInterruptibly()的用法时已经体现了Lock的可中断性。

3.公平锁

　　公平锁即尽量以请求锁的顺序来获取锁。比如同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该所，这种就是公平锁。

　　非公平锁即无法保证锁的获取是按照请求锁的顺序进行的。这样就可能导致某个或者一些线程永远获取不到锁。

　　在Java中，synchronized就是非公平锁，它无法保证等待的线程获取锁的顺序。

　　而对于ReentrantLock和ReentrantReadWriteLock，它默认情况下是非公平锁，但是可以设置为公平锁。这一点由构造函数可知：

4.读写锁

　　读写锁将对一个资源（比如文件）的访问分成了2个锁，一个读锁和一个写锁。

　　正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。

　　ReadWriteLock就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。

　　可以通过readLock()获取读锁，通过writeLock()获取写锁。

	问题1： Thread.interrupt()方法和InterruptedException异常的关系？是由interrupt触发产生了InterruptedException异常？ 
答： Thread.interrupt()只是在Object.wait() .Object.join(), Object.sleep()几个方法会主动抛出InterruptedException异常。而在其他的的block常见，只是通过设置了Thread的一个标志位信息，需要程序自我进行处理
	问题2：Thread.interrupt()会中断线程什么状态的工作？ RUNNING or BLOCKING？ 
答：Thread.interrupt设计的目的主要是用于处理线程处于block状态，比如wait(),sleep()状态就是个例子。但可以在程序设计时为支持task cancel，同样可以支持RUNNING状态。比如Object.join()和一些支持interrupt的一些nio channel设计。 
	问题3： 一般Thread编程需要关注interrupt中断不？一般怎么处理？可以用来做什么？ 
答： interrupt用途： unBlock操作，支持任务cancel， 数据清理等。 
	问题4： LockSupport.park()和unpark()，与object.wait()和notify()的区别？ 
答： 
1. 面向的主体不一样。LockSuport主要是针对Thread进进行阻塞处理，可以指定阻塞队列的目标对象，每次可以指定具体的线程唤醒。Object.wait()是以对象为纬度，阻塞当前的线程和唤醒单个(随机)或者所有线程。 
2. 实现机制不同。虽然LockSuport可以指定monitor的object对象，但和object.wait()，两者的阻塞队列并不交叉。可以看下测试例子。object.notifyAll()不能唤醒LockSupport的阻塞Thread. 
	问题5： LockSupport.park(Object blocker)传递的blocker对象做什么用？ 
答: 对应的blcoker会记录在Thread的一个parkBlocker属性中,通过jstack命令可以非常方便的监控具体的阻塞对象. 
	问题6： LockSupport能响应Thread.interrupt()事件不？会抛出InterruptedException异常？ 
答：能响应interrupt事件，但不会抛出InterruptedException异常。针对LockSupport对Thread.interrupte支持


JavaAPI对LockSupport的解释是：用来创建锁和其他同步类的基本线程阻塞原语。
LockSupport 与Thread.suspend()和Thread.resume()的区别
在LockSupport出现之前，如果要block/unblock某个Thread，除了使用Java语言内置的monitor机制之外，只能通过Thread.suspend()和Thread.resume()。目前这两个方法都被标注为废弃，为什么 Thread.suspend 和 Thread.resume 被废弃了？
来自Oracle的官方文档Why Are Thread.stop, Thread.suspend, Thread.resume and Runtime.runFinalizersOnExit Deprecated?有如下解释：
Thread.suspend 天生容易引起死锁。如果目标线程挂起时在保护系统关键资源的监视器上持有锁，那么其他线程在目标线程恢复之前都无法访问这个资源。如果要恢复目标线程的线程在调用 resume 之前试图锁定这个监视器，死锁就发生了。这种死锁一般自身表现为“冻结（ frozen ）”进程。
LockSupport使用park和unpark方法进行线程的阻塞和解除阻塞操作，每个使用这个类的线程都将与一个许可关联，如果许可可用，则调用park立即返回；否则可能阻塞。如果许可不可用，则调用unpark使其可用，从而避免了死锁问题。

LockSupport 与 Object Monitor的区别
=====================================
LockSupport是针对特定线程来进行阻塞和解除阻塞操作的；而Object的wait()/notify()/notifyAll()是用来操作特定对象的等待集合的。有关Java内置monitor的知识可参考：Java Monitor 理论与实践。
LockSupport与伪唤醒问题
跟Object.wait()方法一样，park系列方法也会因为伪唤醒的原因返回。为了防止上述情况发生，park方法的使用需要遵守如下约定：

while (!canProceed()) { ... LockSupport.park(this); }

============================================================================
AbstractOwnableSynchronizer.class，这三个AbstractXXXSynchronizer都是为了创建锁和相关的同步器而提供的基础，锁，还有前面提到的同步设备都借用了它们的实现逻辑
AbstractQueuedLongSynchronizer.class，AbstractOwnableSynchronizer的子类，所有的同步状态都是用long变量来维护的，而不是int，在需要64位的属性来表示状态的时候会很有用
AbstractQueuedSynchronizer.class，为实现依赖于先进先出队列的阻塞锁和相关同步器（信号量、事件等等）提供的一个框架，它依靠int值来表示状态
Lock.class，Lock比synchronized关键字更灵活，而且在吞吐量大的时候效率更高，根据JSR-133的定义，它happens-before的语义和synchronized关键字效果是一模一样的，它唯一的缺点似乎是缺乏了从lock到finally块中unlock这样容易遗漏的固定使用搭配的约束，除了lock和unlock方法以外，还有这样两个值得注意的方法：
lockInterruptibly：如果当前线程没有被中断，就获取锁；否则抛出InterruptedException，并且清除中断
tryLock，只在锁空闲的时候才获取这个锁，否则返回false，所以它不会block代码的执行
ReadWriteLock.class，读写锁，读写分开，读锁是共享锁，写锁是独占锁；对于读-写都要保证严格的实时性和同步性的情况，并且读频率远远大过写，使用读写锁会比普通互斥锁有更好的性能。
ReentrantLock.class，可重入锁（lock行为可以嵌套，但是需要和unlock行为一一对应），有几点需要注意：
构造器支持传入一个表示是否是公平锁的boolean参数，公平锁保证一个阻塞的线程最终能够获得锁，因为是有序的，所以总是可以按照请求的顺序获得锁；不公平锁意味着后请求锁的线程可能在其前面排列的休眠线程恢复前拿到锁，这样就有可能提高并发的性能
还提供了一些监视锁状态的方法，比如isFair、isLocked、hasWaiters、getQueueLength等等
ReentrantReadWriteLock.class，可重入读写锁
Condition.class，使用锁的newCondition方法可以返回一个该锁的Condition对象，如果说锁对象是取代和增强了synchronized关键字的功能的话，那么Condition则是对象wait/notify/notifyAll方法的替代。在下面这个例子中，lock生成了两个condition，一个表示不满，一个表示不空；在put方法调用的时候，需要检查数组是不是已经满了，满了的话就得等待，直到“不满”这个condition被唤醒（notFull.await()）；在take方法调用的时候，需要检查数组是不是已经空了，如果空了就得等待，直到“不空”这个condition被唤醒（notEmpty.await()）：


Atomic原子类：CAS原理，基本类，引用类，数组类，属性的原子修改器
===============================================================
基本类：AtomicInteger、AtomicLong、AtomicBoolean；
引用类型：AtomicReference、AtomicStampedRerence、AtomicMarkableReference；
数组类型：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray
属性原子修改器（Updater）：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater

public final boolean compareAndSet(int expect, int update) {   
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
 }

整体的过程就是这样子的，利用CPU的CAS指令，同时借助JNI来完成Java的非阻塞算法。其它原子操作都是利用类似的特性完成的。 
而整个J.U.C都是建立在CAS之上的，因此对于synchronized阻塞算法，J.U.C在性能上有了很大的提升。 
这些对象都的行为在不使用同步的情况下保证了原子性。值得一提的有两点：
weakCompareAndSet方法：compareAndSet方法很明确，但是这个是啥？根据JSR规范，调用weakCompareAndSet时并不能保证happen-before的一致性，因此允许存在重排序指令等等虚拟机优化导致这个操作失败（较弱的原子更新操作），但是从Java源代码看，它的实现其实和compareAndSet是一模一样的；
lazySet方法：延时设置变量值，这个等价于set方法，但是由于字段是volatile类型的，因此次字段的修改会比普通字段（非volatile字段）有稍微的性能损耗，所以如果不需要立即读取设置的新值，那么此方法就很有用。
AtomicBoolean.class
AtomicInteger.class
AtomicIntegerArray.class
AtomicIntegerFieldUpdater.class
AtomicLong.class
AtomicLongArray.class
AtomicLongFieldUpdater.class
AtomicMarkableReference.class，它是用来高效表述Object-boolean这样的对象标志位数据结构的，一个对象引用+一个bit标志位
AtomicReference.class
AtomicReferenceArray.class
AtomicReferenceFieldUpdater.class
AtomicStampedReference.class，它和前面的AtomicMarkableReference类似，但是它是用来高效表述Object-int这样的“对象+版本号”数据结构，特别用于解决ABA问题

Volatile用法
============
	在多线程并发编程中synchronized和Volatile都扮演着重要的角色，Volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。它在某些情况下比synchronized的开销更小，本文将深入分析在硬件层面上Intel处理器是如何实现Volatile的，通过深入分析能帮助我们正确的使用Volatile变量。

	有volatile变量修饰的共享变量进行写操作的时候会多第二行汇编代码，通过查IA-32架构软件开发者手册可知，lock前缀的指令在多核处理器下会引发了两件事情。
	将当前处理器缓存行的数据会写回到系统内存。
	这个写回内存的操作会引起在其他CPU里缓存了该内存地址的数据无效。

synchronized 关键字
====================
synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种： 
1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象； 
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象； 
3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象； 
4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。

synchronized(Sync.class)实现了全局锁的效果。

static synchronized方法，static方法可以直接类名加方法名调用，方法中无法使用this，所以它锁的不是this，而是类的Class对象，所以，static synchronized方法也相当于全局锁，相当于锁住了代码段。

A. 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类，该类所有的对象同一把锁。 
B. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。 
C. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。

ThreadLocal不是用来解决对象共享访问问题的，而主要是提供了保持对象的方法和避免参数传递的方便的对象访问方式。归纳了两点：
=============
         1、每个线程中都有一个自己的ThreadLocalMap类对象，可以将线程自己的对象保持到其中，各管各的，线程可以正确的访问到自己的对象。 

         2、将一个共用的ThreadLocal静态实例作为key，将不同对象的引用保存到不同线程的ThreadLocalMap中，然后在线程执行的各处通过这个静ThreadLocal实例的get()方法取得自己线程保存的那个对象，避免了将这个对象作为参数传递的麻烦。 
         
Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。
```