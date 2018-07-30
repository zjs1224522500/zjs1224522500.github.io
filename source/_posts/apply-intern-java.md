---
title: '2017.11找实习-面经-Java'
date: 2017-12-21 20:13:04
tags: 面经
---

> ##### 2017年11月，为了2018年上半年的企业实习投递了一系列简历并参与了相关面试。  

---
> ###### 主要分为Java篇，数据库篇，网络篇，框架篇，算法篇和相关工具篇

---
> - 面试的岗位主要是Java后端开发实习生的岗位.
> - 主要面的有今日头条、百度、网易、搜狐、滴滴、SAP
> - 较为简单的有所省略直接给相关问题或者直接上链接
> - 较为常见的或者说比较重要的后续会单独写博文或者贴一些友链进行探讨
> - 因为确实面试相关的博客已经很多很多，我想做的其实更多的是一个汇总
> ---
> - 另外，不同岗位的直接看CS公共基础知识部分的即可（OS、计网、计组）
> #### 此篇为Java篇，持续更新ing

## 1、Java基本知识
### 实现多线程的方式
> **之后会统一的进行并发编程的相关知识点的整理和总结**
- 常见方式一：继承 Thread 类；对应的重写run()方法，start() 方法执行
- 常见方式二：实现 Runable 接口；对应的重写run()方法，start() 方法执行
- 方式三：实现Callable接口通过FutureTask包装器来创建Thread线程
- 方式四：使用ExecutorService、Callable、Future实现有返回结果的线程
- [参考博文：Java中的多线程你只要看这一篇就够了](https://www.cnblogs.com/wxd0108/p/5479442.html)
- [参考博文：JAVA多线程实现的四种方式](https://www.cnblogs.com/felixzh/p/6036074.html)


<!-- more -->
### list、map、set的区别和应用场景（集合框架）
###### 1.存放
- (1)List存放元素是有序，可重复，继承Collection
- (2)Set存放元素无序，不可重复，继承Collection
```java
public interface Collection<E> extends Iterable<E> {
    int size();
    boolean isEmpty();
    boolean contains()
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    // 取交集
    boolean retainAll(Collection<?> c);
    
    Object[] toArray();
    <T> T[] toArray(T[] a);
    
    Iterator<E> iterator();

    // ...
}
```
- (3)Map元素键值对形式存放，键无序不可重复，值可重复
```java
public interface Map<K,V> {
    int size();
    boolean isEmpty();
    boolean containsKey(Object key);
    boolean containsValue(Object value);
    V get(Object key);
    V put(K key, V value);
    V remove(Object key);
    void putAll(Map<? extends K, ? extends V> m);
    void clear();
    Set<K> keySet();
    Collection<V> values();
    Set<Map.Entry<K, V>> entrySet();
    //...
}
```

###### 2.取出 
> **Java8** 可使用 **lamda表达式** 以及  **函数式编程** 实现访问集合中元素，此处暂不涉及，后续会统一对 **Java8** 的相关特性进行讲解。 

- (1)List取出元素for循环，foreach循环，Iterator迭代器迭代
```java
// 使用 for 循环
public void printAllUsingFor(List<E> list) {
	// 注意将size() 的调用尽可能减少调用次数并使用临时空间变量节省空间
	for (int i = 0, size = list.size(); i < size; i++) {
		System.out.print(list.get(i));
	}
	System.out.println();
}

// 使用 foreach （其本质仍为迭代器，Java编译器在生成字节码时对应的修改为迭代器）
public void printAllUsingForEach(List<E> list) {
	for (E e : list) {
		System.out.print(e);
	}
	System.out.println();
}

// 使用 迭代器
public void printAllUsingIterator(List<E> list) {
	for (Iterator iterator = list.iterator(); iterator.hasNext(); ) {
		E e = (E) iterator.next();
		System.out.print(e);
	}
	System.out.println();
}

// 最佳实践（list中存放的元素数目较少时无论采用哪种遍历方式，结果都相差不大
// 但当数据量相对较大时，则需要根据list的类型对应的进行遍历方式的调整。
public void printAll(List<E> list) {
	if (list instanceof RandomAccess) {
		printAllUsingFor(list);
	} else {
		printAllUsingIterator(list);
	}
}
```

- (2)Set取出元素foreach循环，Iterator迭代器迭代
```java
// 迭代器 与 List的迭代器访问同理，此处不再演示
// 同时 迭代器 与 foreach 效率相差无几，不再具体进行相关对比
public void printAllUsingForEach(Set<E> set) {
	for (E e : set) {
		System.out.print(e);
	}
	System.out.println();
}
```

- (3)Map取出元素需转换为Set，然后进行Iterator迭代器迭代，或转换为Entry对象进行Iterator迭代器迭代 
```java
// 遍历 key 和 value，对应的也可以根据key去获取value
public void printKeySetAndValueSet(Map<E, Object> map) {
	for (E e : map.keySet()) {
		System.out.println("keys = " + e);
	}

	for (Object value : map.values()) {
		System.out.println("values = " + value);
	}
}

// foreach
public void printAllUsingForEach(Map<E, Object> map) {
	for (Map.Entry<E, Object> entry : map.entrySet()) {
		System.out.println("key = " + entry.getKey() + ";value = " + entry.getValue());
	}
}

// iterator 两种方式（是否使用泛型）
public void printAllUsingIterator(Map<E, Object> map) {
	for (Iterator iterator = map.entrySet().iterator(); iterator.hasNext(); ) {
		// 需要强转，遍历器返回的类型为 Object
		Map.Entry<E, Object> entry = (Map.Entry) iterator.next();
		System.out.println("key = " + entry.getKey() + "; value = " + entry.getValue());
	}
	
	for (Iterator<Map.Entry<E, Object>> iterator = map.entrySet().iterator(); iterator
				.hasNext(); ) {
		// 不需要强转，由于返回类型同 获得的遍历器 泛型一致
		Map.Entry<E, Object> entry = iterator.next();
		System.out.println("key = " + entry.getKey() + "; value = " + entry.getValue());
	}
}


```

> [相关博文：Java迭代器的使用以及和for的区别](http://blog.csdn.net/u012442401/article/details/47850501)


#### List中的相关集合框架（ArrayList、LinkedList、Stack、Vector）
##### linkedList和arrayList的区别
> 区别最主要是 **顺序存储** 和 **链式存储** 的区别

- 1．对ArrayList和LinkedList而言，在列表末尾增加一个元素所花的开销都是固定的。对ArrayList而言，主要是在内部数组中增加一项，指向所添加的元素，偶尔可能会导致对数组重新进行分配；而对LinkedList而言，这个开销是统一的，分配一个内部Entry对象。

- 2．在ArrayList的中间插入或删除一个元素意味着这个列表中剩余的元素都会被移动；而在LinkedList的中间插入或删除一个元素的开销是固定的。

- 3．LinkedList不支持高效的随机元素访问。

- 4．ArrayList的空间浪费主要体现在在list列表的结尾预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗相当的空间

> 可以这样说：当操作是在一列数据的后面添加数据而不是在前面或中间,并且需要随机地访问其中的元素时,使用ArrayList会提供比较好的性能；当你的操作是在一列数据的前面或中间添加或删除数据,并且按照顺序访问其中的元素时,就应该使用LinkedList了。 

##### ArrayList、Stack和Vector的主要区别
- 1、Vector、Stack：线程安全；ArrayList、LinkedList：非线程安全。**对于相关线程安全实现的机制将在并发编程中进行统一的讲解！**
- 2、实现方式： LinkedList：双向链表，ArrayList，Vector，Stack：数组；
- 3、Stack继承自Vector，只在Vector的基础上添加了几个Stack相关的方法
```java
public class Stack<E> extends Vector<E> {
    public E push(E item) {
        // public synchronized void addElement(E obj)
        addElement(item);
        return item;
    }
    
    public synchronized E pop() {...}
    public synchronized E peek() {...}
    
    public boolean empty() {
        return size() == 0;
    }
    
    public synchronized int search(Object o) {
        int i = lastIndexOf(o);
        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }
    
    ...
}
```
- 4、ArrayList和Vector在动态扩容时有所不同。
```java
// ArrayList
int newCapacity = (oldCapacity * 3)/2 + 1;  

// Vector
int newCapacity = (capacityIncrement > 0) ?  
            (oldCapacity + capacityIncrement) : (oldCapacity * 2);  

```
#### Set相关的集合框架（HashSet、TreeSet）
###### HashSet 和 TreeSet
- 1.HashSet 是无序的，不能保证元素排列顺序，底层是由 HashMap 进行实现的。**（讲HashMap时会进行具体阐述）**，会根据对应的hashCode进行相关排序；TreeSet 是有序的，底层是通过 TreeMap 实现 **（对应的会单独讲实现原理---红黑树 Entry）** 的。TreeSet 通过使用 compareTo() 方法来比较然后升序排列，也可以定制排序逻辑。
```java
TreeSet treeSet = new TreeSet();
HashSet hashSet = new HashSet();
hashSet.add(1);
hashSet.add(0);
hashSet.add(3);
hashSet.add(2);
hashSet.add(-1);
System.out.println(hashSet);// >> [0, -1, 1, 2, 3]

treeSet.add(1);
treeSet.add(0);
treeSet.add(3);
treeSet.add(2);
treeSet.add(-1);
System.out.println(treeSet);// >> [-1, 0, 1, 2, 3]
```

- 2、HashSet 允许存放 null，但只能存放一个 null **（由于Hash表的原因）**，而 TreeSet 不能，会抛出对应的空指针异常。
```java
hashSet.add(1);
hashSet.add(null);
hashSet.add(null);
System.out.println(hashSet); // >> [null,1]
```

###### hashMap的原理
- 会新开博文具体阐释三者原理和区别
- [参考友链：HashMap实现原理分析](http://blog.csdn.net/vking_wang/article/details/14166593)
###### hashTable的原理
- 会新开博文结合源码具体阐释三者原理和区别
- [参考博文：深入Java集合学习系列：Hashtable的实现原理](http://blog.csdn.net/zheng0518/article/details/42199477)
###### concurrentHashMap的原理
- 会新开博文具体阐释三者原理和区别
- [参考友链：ConcurrentHashMap原理分析](http://www.importnew.com/16142.html)


#### Object中常用的方法
##### hashCode
- Object 类会提供 hashCode() 方法的声明并使用 native 关键字标志其由非java语言实现，具体的方法实现在外部。
```java
public native int hashCode();
```

- String 中的 hashCode()。hash 对应的算法.  
[相关博文：关于hashcode 里面 使用31 系数的问题](http://blog.csdn.net/tayanxunhua/article/details/20525251)

![image](http://chart.googleapis.com/chart?cht=tx&chl=s%5B0%5D*31%5E%7Bn-1%7D%2Bs%5B1%5D*31%5E%7Bn-2%7D%2B...%2Bs%5Bn-1%5D)

- String类是用它的value值作为参数来计算hashCode的，也就是说，相同的value就一定会有相同的hashCode值。但反之不成立，即hashCode相同，value值不一定相同。这种情况又被称为**hash冲撞（冲突）**,hashMap中对应的解决方式就是hashCode相同的对象再构造一个线性表。
```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
- 当然最好的 **hashCode()** 方法则是所有不同的对象都能有各自的 **Hash码**，同时注意在设计对应的 **hashCode()** 方法时，
> 无论何时，对同一个对象调用hashCode()都应该产生同样的值。如果在讲一个对象用put()添加进HashMap时产生一个hashCdoe值，而用get()取出时却产生了另一个hashCode值，那么就无法获取该对象了。所以如果你的hashCode方法依赖于对象中易变的数据，用户就要当心了，因为此数据发生变化时，hashCode()方法就会生成一个不同的散列码.---**By Effective Java**

- [参考博文：重写HashCode的内存变化过程以及两种重写hashCode方式的比较](http://blog.csdn.net/benjaminzhang666/article/details/9468605)
- [参考博文：通用的Java hashCode重写方案](http://blog.csdn.net/sunmenggmail/article/details/18660699)

##### equals 和 ==
[参考博文：equals和==的区别小结](https://www.cnblogs.com/Eason-S/p/5524837.html)
###### == 
- == 比较的是变量(栈)内存中存放的对象的(堆)内存地址，用来判断两个对象的地址是否相同，即是否是指相同一个对象。比较的是真正意义上的指针操作。

###### equals()
- equals用来比较的是两个对象的内容是否相等，由于所有的类都是继承自java.lang.Object类的，所以适用于所有对象，如果没有对该方法进行覆盖的话，调用的仍然是Object类中的方法，而Object中的equals方法返回的却是==的判断。
- 默认的 equals() 和 String 中的 equals()
```java
public boolean equals(Object obj) {
    return (this == obj);
}

public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = count;
        if (n == anotherString.count) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = offset;
            int j = anotherString.offset;
            while (n-- != 0) {
                if (v1[i++] != v2[j++])
                    return false;
            }
            return true;
        }
    }
    return false;
}
```
- 对于自定义的对象，当重写 equals() 方法时需要对应的重写 hashCode() 方法
    - 1、如果两个对象相同（即用equals比较返回true），那么它们的hashCode值一定要相同；
    - 2、如果两个对象的hashCode相同，它们并不一定相同(即用equals比较返回false)
- 考虑到 Java 的集合框架中有很多关于 Hash 表的相关实现。hashCode() 的效率相对于复杂的业务逻辑对应的 equals() 方法而言是要高得多的。所以在相关数据结构中采用先 hash 的方式再利用 equals() 进行相关比较能够更好的提高性能，同时保证 hash 表中存放的元素不相同。


###### 应用场景
- 1、原生类型如:int/char/boolean等使用 **==** 进行比较，自定义的对象使用 **equals()**
- 2、**==** 返回true如果两个引用指向相同的对象，**equals()** 的返回结果依赖于具体业务实现
- 3、字符串的对比使用 **equals()** 代替 **==** 操作符

##### clone
- [参考博文：详解Java中的clone方法 -- 原型模式](http://blog.csdn.net/zhangjg_blog/article/details/18369201)
- [参考博文：java之clone方法的使用](http://blog.csdn.net/rebirth_love/article/details/51792014)
- 浅拷贝 和 深拷贝 的区别此处不再赘述。其实是 引用复制 和 内存复制的区别
- 用法：实现对应的Cloneable接口，重写对应 clone() 方法，并调用父类 super.clone()
- **注意**：完全意义上的深拷贝几乎是不存在的如果在拷贝一个对象时，要想让这个拷贝的对象和源对象完全彼此独立，那么在引用链上的每一级对象都要被显式的拷贝。所以创建彻底的深拷贝是非常麻烦的，尤其是在引用关系非常复杂的情况下， 或者在引用链的某一级上引用了一个第三方的对象，而这个对象没有实现clone方法， 那么在它之后的所有引用的对象都是被共享的。

##### toString
- 若自定义的对象未重写 **toString()** 方法，则会调用父类 **Object** 对应的 **toString()** 输出对应的 **对象的地址**.
```java
public String toString() {
   return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
- 调用 print相关函数时，会默认调用 toString()
```java
public void println(Object x) {
    String s = String.valueOf(x);
    synchronized (this) {
        print(s);
        newLine();
    }
}
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
```
#### Java程序从编写到运行的全过程：
##### 简单描述
- 1、编写代码
- 2、编译，Java 文件 编译为 class 文件
- 3、类加载 ClassLoader为执行程序寻找和装载所需要的类。
- 4、字节码校验。对class文件的代码进行校验，保证代码的安全性。
- 5、解释（Interpreter）。解释器解释class文件转为机器码。
- 6、运行：由运行环境中的Runtime对代码进行运行
- [参考博文：简述Java 从代码到运行的全过程](http://blog.csdn.net/cjj3930337/article/details/6905873)
- [参考博文：Java代码编译过程简述](http://blog.csdn.net/fuzhongmin05/article/details/54880257)

##### 之后会结合Java内存结构进行具体阐释。敬请期待


