title: 校招复习-HashMap
date: 7/22/2016 4:34:49 PM   
categories: CampusRecrutmentPrepare
---


#HashMap的实现原理
- HashMap是一个数组和链表的结合体（在数据结构称“链表散列“）
	- 当我们往hashmap中put元素的时候，先根据key的hash值得到这个元素在数组中的位置（即下标），然后就可以把这个元素放到对应的位置中了。
	  如果这个元素所在的位子上已经存放有其他元素了，那么在同一个位子上的元素将以链表的形式存放，新加入的放在链头，最先加入的放在链尾。

- 实现简单原理
> HashMap 在底层将 key-value 当成一个整体进行处理，这个整体就是一个 Entry 对象。HashMap 底层采用一个 Entry[] 数组来保存所有的 key-value 对，当需要存储一个 Entry 对象时，会根据 Hash 算法来决定其存储位置；当需要取出一个 Entry 时，也会根据 Hash 算法找到其存储位置，直接取出该 Entry。由此可见：HashMap 之所以能快速存、取它所包含的 Entry，完全类似于现实生活中母亲从小教我们的：不同的东西要放在不同的位置，需要时才能快速找到它

> HashMap 类的 put(K key , V value) 方法的源代码
	
	public V put(K key, V value)   
	{   
	 // 如果 key 为 null，调用 putForNullKey 方法进行处理  
	 if (key == null)   
	     return putForNullKey(value);   
	 // 根据 key 的 keyCode 计算 Hash 值  
	 int hash = hash(key.hashCode());   
	 // 搜索指定 hash 值在对应 table 中的索引  
	     int i = indexFor(hash, table.length);  
	 // 如果 i 索引处的 Entry 不为 null，通过循环不断遍历 e 元素的下一个元素  
	 for (Entry<K,V> e = table[i]; e != null; e = e.next)   
	 {   
	     Object k;   
	     // 找到指定 key 与需要放入的 key 相等（hash 值相同  
	     // 通过 equals 比较放回 true）  
	     if (e.hash == hash && ((k = e.key) == key   
	         || key.equals(k)))   
	     {   
	         V oldValue = e.value;   
	         e.value = value;   
	         e.recordAccess(this);   
	         return oldValue;   
	     }   
	 }   
	 // 如果 i 索引处的 Entry 为 null，表明此处还没有 Entry   
	 modCount++;   
	 // 将 key、value 添加到 i 索引处  
	 addEntry(hash, key, value, i);   
	 return null;   
	}   

> 根据上面 put 方法的源代码可以看出，当程序试图将一个 key-value 对放入 HashMap 中时，程序首先根据该 key 的 hashCode() 返回值决定该 Entry 的存储位置：如果两个 Entry 的 key 的 hashCode() 返回值相同，那它们的存储位置相同。如果这两个 Entry 的 key 通过 equals 比较返回 true，新添加 Entry 的 value 将覆盖集合中原有 Entry 的 value，但 key 不会覆盖。如果这两个 Entry 的 key 通过 equals 比较返回 false，新添加的 Entry 将与集合中原有 Entry 形成 Entry 链，而且新添加的 Entry 位于 Entry 链的头部——具体说明继续看 addEntry() 方法的说明。 

- HashMap的构造
> 当创建一个 HashMap 时，系统会自动创建一个 table 数组来保存 HashMap 中的 Entry，下面是 HashMap 中一个构造器的代码： 

	// 以指定初始化容量、负载因子创建 HashMap   
	public HashMap(int initialCapacity, float loadFactor)   
	{   
		 // 初始容量不能为负数  
		 if (initialCapacity < 0)   
		     throw new IllegalArgumentException(   
		    "Illegal initial capacity: " +   
		         initialCapacity);   
		 // 如果初始容量大于最大容量，让出示容量  
		 if (initialCapacity > MAXIMUM_CAPACITY)   
		     initialCapacity = MAXIMUM_CAPACITY;   
		 // 负载因子必须大于 0 的数值  
		 if (loadFactor <= 0 || Float.isNaN(loadFactor))   
		     throw new IllegalArgumentException(   
		     loadFactor);   
		 // 计算出大于 initialCapacity 的最小的 2 的 n 次方值。  
		 int capacity = 1;   
		 while (capacity < initialCapacity)   
		     capacity <<= 1;   
		 this.loadFactor = loadFactor;   
		 // 设置容量极限等于容量 * 负载因子  
		 threshold = (int)(capacity * loadFactor);   
		 // 初始化 table 数组  
		 table = new Entry[capacity];            // ①  
		 init();   
	}   

> 对于HashMap的容量是这样确定的：找出大于 initialCapacity 的、最小的 2 的 n 次方值，并将其作为 HashMap 的实际容量（由 capacity 变量保存）。

> 当创建 HashMap 时，有一个默认的负载因子（load factor），其默认值为 0.75，这是时间和空间成本上一种折衷：增大负载因子可以减少 Hash 表（就是那个 Entry 数组）所占用的内存空间，但会增加查询数据的时间开销，而查询是最频繁的的操作（HashMap 的 get() 与 put() 方法都要用到查询）；减小负载因子会提高数据查询的性能，但会增加 Hash 表所占用的内存空间。 

> 掌握了上面知识之后，我们可以在创建 HashMap 时根据实际需要适当地调整 load factor 的值；如果程序比较关心空间开销、内存比较紧张，可以适当地增加负载因子；如果程序比较关心时间开销，内存比较宽裕则可以适当的减少负载因子。通常情况下，程序员无需改变负载因子的值。

- 元素的保存，以及冲突解决（生成链）
>- 当系统开始初始化 HashMap 时，系统会创建一个长度为 capacity 的 Entry 数组，这个数组里可以存储元素的位置被称为“桶（bucket）”，每个 bucket 都有其指定索引，系统可以根据其索引快速访问该 bucket 里存储的元素。 
>- 无论何时，HashMap 的每个“桶”只存储一个元素（也就是一个 Entry），由于 Entry 对象可以包含一个引用变量（就是 Entry 构造器的的最后一个参数）用于指向下一个 Entry，因此可能出现的情况是：HashMap 的 bucket 中只有一个 Entry，但这个 Entry 指向另一个 Entry ——这就形成了一个 Entry 链。


- HashMap的读取
> 当 HashMap 的每个 bucket 里存储的 Entry 只是单个 Entry ——也就是没有通过指针产生 Entry 链时，此时的 HashMap 具有最好的性能：当程序通过 key 取出对应 value 时，系统只要先计算出该 key 的 hashCode() 返回值，在根据该 hashCode 返回值找出该 key 在 table 数组中的索引，然后取出该索引处的 Entry，最后返回该 key 对应的 value 即可