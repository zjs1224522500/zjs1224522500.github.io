---
title: 'Java基本常识'
date: 2017-04-30 00:55:13
tags: JavaSE
---
### Java体系架构
> JavaSE : Java Standard Edition  
> JavaEE : Java Enterprise Edition  
> JavaME : Java Micro Edition  

### Java语言的特点
* 面向对象  
* 跨平台（提供了在不同平台下运行的解释环境）
* 健壮性（吸收了C/C++的特点）
* 安全性较高（自动回收垃圾，强制类型检查，取消指针）

### Java跨平台的原理
* Java源代码 .java 编译为 Java字节码 .class（跨平台）
* Java字节码 .class 执行在 Java虚拟机 JVM
* 虚拟机JVM 基于不同操作系统（MAC、Linux、Windows)
* .class 文件由不同操作系统的虚拟机解释给操作系统。
<!-- more -->
### JVM Java虚拟机
![image](http://img.my.csdn.net/uploads/201212/21/1356070838_2499.png)
* .class 对应的字节码在虚拟机上解释器再次进行“解释”和即使编译器上进行即时编译，再到达运行期系统，再到操作系统，再到硬件
* JVM 可以理解成一个可运行Java字节码的虚拟计算系统，包含一个解释器组件，可以实现Java字节码和计算机操作系统之间的通信，对于不同的运行平台，有不同的JVM
* JVM屏蔽了底层运行平台的差别，实现了“一次编译，随处运行”。

### 垃圾回收器
* 不再使用的内存空间应当进行回收 - 垃圾回收
* JVM 提供了一种系统线程跟踪存储空间的分配情况。
* 并在JVM的空闲时，检查并释放那些可以被释放的存储空间，自动运行。

### JavaSE 的组成概念图
![image](http://img.blog.csdn.net/20151030225424963?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 数组内存结构分析
* 栈内存（先进后出，栈内存大小是固定的）
> 存储确定大小的数据类型：临时变量，基本数据类型，引用类型。由于固定大小，所以存取速度较快。
* 堆内存 
> 存储不确定大小的值（譬如值可能发生改变），需要临时地动态地进行分配，速度相对于栈较慢。  

``` 
    譬如:  
    String[] name = {"AA","bb","CC"};
    name 这个数组名会在栈内存中开辟空间，name 对应的是数组的地址
    而 name 数组所对应的值则会在堆内存中开辟空间
```

### 多维数组
- 数组动态扩展,当数组要满时进行扩展
```java
    int[] array = new int[3];
    int newlen = array.length*3/2 + 1;
    array = Arrays.copyOf(array,newlen);
```


### 对象与内存分析
#### new 关键字深入
* 1. new 关键字表示创建一个对象
* 2. new 关键字表示实例化对象
* 3. new 关键字表示申请内存空间

#### 对象内存分析
```java
class Dog{
    String name;
    int age;
}
```
- 栈内存中将储存对象的内存地址
![这里写图片描述](http://img.blog.csdn.net/20170723132726810?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 给对象的属性赋值
![这里写图片描述](http://img.blog.csdn.net/20170723132750618?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 在内存中创建多个对象
![这里写图片描述](http://img.blog.csdn.net/20170723132809732?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
- 给多个对象属性赋值
![这里写图片描述](http://img.blog.csdn.net/20170723132827149?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 声明两个对象，一个实例化，一个没有实例化
![这里写图片描述](http://img.blog.csdn.net/20170723132847790?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 对象之间的赋值（只要类型相同就可以进行赋值）
![这里写图片描述](http://img.blog.csdn.net/20170723132901520?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 分别实例化两个对象
![这里写图片描述](http://img.blog.csdn.net/20170723132919348?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 此时对象之间的赋值（此时虚拟机将进行自动垃圾回收 GC ）
![这里写图片描述](http://img.blog.csdn.net/20170723132938235?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    - 此时栈中没有对象指向堆中这一块内存，位于堆中的这一块内存将再也无法获取到，成为垃圾。

### 构造方法
- 构造方法是在类中定义的，构造方法的定义格式：方法名称与类名称相同，无返回类型的声明。
- 对象在实例化时，常常是（以无参构造方法为例，有参构造方法同理）：

> Type name = new Type();

此时的 Type() 其实就是调用了相应的构造方法。

### 方法重载 ovrloading method
- 在类中可以创建多个方法，它们具有相同的名字，但具有不同的参数和不同的定义。






### 匿名对象
- 匿名对象就是定义一个没有名称的对象
- 该对象的特点是只能使用一次
- 该对象会直接在堆中开辟内存空间
- 该对象被使用后会成为垃圾对象，被GC回收
```java
class Dog{
    private String name;
    private int age;
    public say();
}

// 该对象只能使用一次
new Dog().say();
```

### 值传递与引用传递

- 示例：值传递
```java

int x = 10;
method(x);
System.out.println("x="+x);//10

public static void method(int mx){
    mx = 20;
}
```
- 内存分析
![这里写图片描述](http://img.blog.csdn.net/20170723133751680?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 示例：引用传递
```java

Cat c = new Cat();
method(c);
System.out.println("Cat age = " + c.age);//5

public static void method(Cat cat){
    cat.age = 5;
}

class Cat{
    int age = 2;
}
```

- 内存分析
![这里写图片描述](http://img.blog.csdn.net/20170723132957372?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 示例：String 传递
```java

String name = "小白"；
method(name);
System.out.println(name);//小白

public static void method(String sname){
    sname = "小红";
}

```

- 内存分析：
![这里写图片描述](http://img.blog.csdn.net/20170723133008358?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


   - 执行 method() 之后
 ![这里写图片描述](http://img.blog.csdn.net/20170723133041718?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


- 示例：String 传递
```java

Cat c = new Cat();
method(c);
System.out.println("Cat name = " + c.name);

public static void method(Cat cat){
    cat.name = "小黑"；
}

class Cat{
    String name = "小白";
}

```

- 内存分析


    - 执行method()后

![这里写图片描述](http://img.blog.csdn.net/20170723133115355?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### static关键字

- java虚拟机内存结构

![image](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1494677971&di=802998d84ccd9ffc9e03cb41c6438fe2&imgtype=jpg&er=1&src=http%3A%2F%2Fs7.sinaimg.cn%2Fmw690%2F004lj3yugy6MgQUfyHs96%26amp%3B690)

- static关键字修饰的静态变量与方法就存储在方法区，static关键字修饰的成分不属于对象，直接属于类，而虚拟机中的方法区则主要是存放类的相关信息。线程共享，存储已经被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等等。
- 对于之前String类定义时，当使用直接赋值的方式时所创建的字符串常量不仅要在堆内存中进行创建，还要在方法区中的常量池中进行创建，每一个类对应一个常量池，对应相应的内存区域。
- 声明为static的方法的限制
    - static方法只能调用static方法
    - static方法只能访问和调用static数据
    - static方法不能以任何形式调用this和super
> 本质：static关键字修饰的属性和方法都是属于类的，并不属于对象，所以static修饰的属性和方法实在对象之前进行创建和加载的，故不能调用和访问属于对象的相关属性（非static修饰的属性和方法）。最好是使用类直接去调用static方法，虽然对象也可以调用static方法，但是static是属于类的

### main方法

- public 公共的    虚拟机需要调用
- static 静态的    虚拟机需要调用
- void   无返回值  虚拟机调用无需返回值
- String[] args 表示参数为字符串数组 长度默认为0（不传参） 但实际可以传参

### 对象数组

### foreach 与可变参数  jdk1.5

- 增强for循环  foreach
```java
for(类型变量名称：数组或集合){
    //输出操作
}
```

- 可变参数，可用数组代替
```
返回值类型 方法名称(数据类型...参数名称){
    //同一个类型的参数传入，数量可变
    //但如果有其他类型的参数，需要将可变参数放在最末
}
```

### 代码块

- 普通代码块
> 直接写在方法中的代码块就是普通代码块


- 构造块
> 构造块是在类中定义的代码块,在构造对象时调用，先于构造方法执行

- 静态块 
> 在类中使用static声明的构造块就是静态块,先于构造块执行。因为静态块是属于类的，而不属于对象，所以在加载的时候最先执行的是静态块。在类加载时执行，只执行一次

```java
public class Demo{

    //构造块
    {
        String str = "构造块";
        System.out.println(str);//此时无法输出，只有在new之后即构造之后才会执行
    }
    
    public Demo(){
        System.out.println("我是构造方法");
    }
  
    //静态块  
    static{
        System.out.println("我是静态块");
    }


    public static void main(String[] args){
    
        Code code1 = new Code();
        Code code2 = new Code();
    
    
        // 普通代码块
        {
            String str ="abc";
            System.out.println(str);
        }
    }
}
```

### 单例设计模式
> 保证一个类仅有一个实例，并提供一个访问它的全局访问点。

1. 构造方法私有化
2. 声明一个本类对象
3. 给外部提供一个静态方法获取对象实例

- 两种实现方式
1. 饿汉式   先生成对象（从一开始就创建对象）
2. 懒汉式   后生成对象（需要用到的时候才创建对象，涉及到多线程安全问题，需要改进）

- 应用：工具类一般设计为单例模式

```
public static void main(String[] args) {
	Singleton1 singleton1 = Singleton1.getInstance();
	singleton1.print();
	Singleton1 singleton12 = Singleton1.getInstance();
        System.out.println(singleton1 == singleton12);//因为是同一个对象，输出为true
		
	Singleton2 singleton2 = Singleton2.getInstance();
	singleton2.print();
	}


/**
 * 单例设计模式
 * 饿汉式	
 */
class Singleton1{
	
	// 定义一个对象并实例化
	private static Singleton1 singleton1 = new Singleton1();
	
	// 构造方法私有化
	private Singleton1(){
		
	}
	
	// 获取实例 的静态方法
	public static Singleton1 getInstance(){
		return singleton1;
	}
	
	public void print(){
		System.out.println("饿汉式-单例设计模式");
	}
}

```

```
/**
 * 单例设计模式
 * 懒汉式（多线程访问时会有安全问题）
 */
class Singleton2{
	
	// 定义一个对象并实例化
	private static Singleton2 singleton2 = null;
	
	// 构造方法私有化
	private Singleton2(){
		
	}
	
	// 获取实例 的静态方法
	public static Singleton2 getInstance(){
		if(singleton2 == null){
			singleton2 = new Singleton2();
		}
		return singleton2;
	}
	
	public void print(){
		System.out.println("懒汉式-单例设计模式");
	}
}
```

### 继承

- 继承是面向对象三大特征之一
- 被继承的类成为父类（超类，基类），继承父类的类成为子类（派生类）
- 继承是指一个对象直接使用另一个对象的属性和方法
- 通过继承可以实现代码重用

- 继承的限制
    - Java只能实现单继承，一个类只能有一个父类
    - 允许多层继承，即一个子类只能有一个父类，父类又可以有自己的父类
    - 继承只能继承非私有的属性和方法（非私有属性修饰符：public、default、protected）
    - 构造方法不能被继承（实例化子类时会先调用父类的构造方法）


### 子类的实例化过程
- 在子类进行实例化操作的时候，首先会让父类进行实例化操作，即调用父类的构造方法，再实例化子类。
- 父类无默认的构造方法时，子类必须显式地调用父类的构造方法，利用super关键字（表示父类的引用），super(param)调用父类构造方法语句必须是第一句（子类构造函数中的第一句）。


### 方法的重写
- 方法的重写（overridingng method）  区分overloading
> 子类不想原封不动地继承父类的方法时，即想对从父类出继承来的方法进行一些自定义地改写，这就需要方法的重写，也称为方法的覆盖

- 方法的重写的特性
    - 发生重写的两个方法返回值、方法名、参数列表必须完全一致
    - 子类抛出的异常不能超过父类相应方法抛出的异常（子类异常不能大于父类异常）
    - 子类方法的访问级别不能低于父类的相应方法的访问级别（子类访问级别不能低于父类的访问级别）子类继承后的方法的限定符范围需要大于父类。


- 属性的重写（意义不大）


### super关键字
- 调用父类中的属性、方法、构造方法


### final关键字
- final修饰类，表示为最终类，不能被继承
- final修饰方法，方法不能再被子类重写
- final修饰一个变量，变量将变为常量，不能在改变，一直保持初值。
    - 可以在在声明的时候赋值。
    - 可以在构造方法中赋值。


- final 修饰一个类型的类，表示内存地址不能变
- final 修饰基本数据类型，表示栈中的值不变
- final 修饰一个对象的时候，表示栈中对象的地址不能变，但地址对应的对象是可以改变的。


### 抽象类
> 很多具有相同特征和行为的对象可以抽象为一个类；很多具有相同特征和行为的类可以抽象成一个抽象类。使用abstract关键字声明

- 抽象类的规则
    - 抽象类可以没有抽象方法，有抽象方法的类必须是抽象类
    - 非抽象类继承抽象类必须实现所有的抽象方法
    - 抽象方法可以有方法实现和属性
    - 抽象类不能被实例化
    - 抽象类不能声明为final

- 抽象方法：只有声明，没有实现

### 接口
> 接口是一组行为的规范、定义、没有实现
> 面向对象设计法则：基于接口编程

```java
interface 接口名称{
    全局常量;
    抽象方法;
}
```

- 接口可以继承多个接口
- 一个类可以实现多个接口
- 抽象类实现接口可以不实现方法
- 接口中的所有方法的访问权限都是public
- 接口中定义的属性都是常量
- 接口不能被实例化


### 多态
- 方法的重载和重写
    - 重载发生在同一个类中，返回值相同，方法名相同，参数列表不同。
    - 重写发生在子类和父类里，两种不同的形态

- 对象的多态性
> 对象多态性是从继承关系中的多个类而来

- 向上转型:将子类实例转为父类实例，可以自动转换
```java
//char的容量比int小，所以可以自动完成
int i = 'a';

父类 父类对象 = 子类实例;//自动转换
Person man = new Man();//父类的引用执行子类对象
Person woman = new Woman();

子类 子类对象 = （子类）父类实例; 强制转换
Man m = (Man)man;//强制转换
Man m1 = (Man)woman;//报类型转换异常，转换失败

//整形（4个字节）比char（2个字节）要大，所以需要强制转换
char c = (char) 97;
```

- 方法的重载与重写就是方法的多态性表现
- 多个子类就是父类中的多种形态
- 父类引用可以指向子类对象、自动转换
- 子类对象指向父类引用需要强制转换（类型不对会报异常）
- 在实际开发中尽量使用父类引用(更利于扩展)

### 强制转换
>  在Java中强制类型转换分为基本数据类型和引用数据类型两种

- 子类可以自动转型为父类，但是父类强制转换为子类时只有当引用类型真正的身份为子类时才会强制转换成功，否则失败。


### instanceof关键字
> 格式：对象 instanceof 类型   返回 booleann类型值
- 判断一个对象是否为某个例的实例，是就返回true，不是就返回false

- 父类设计法则
    - 父类通常情况下都设计为抽象类或接口，其中优先考虑接口，如接口不能满足才考虑抽象类，接口优先
    - 一个具体的类尽可能不去继承另一个具体类，这样的好处是无需检查对象是否为父类的对象

```java
package com.review;
public class InstanceofKeyWordDemo {
	public static void main(String[] args) {
		Person man = new Man();
		say((Man)man);//父类对象需要强制转换为子类对象
		
		Woman woman = new Woman();
		say(woman);
	}
	
//	//static 和 返回类型 的位置不可交换
//	public static void say(Man man){
//		man.say();
//	}
//	
//	public static void say(Woman woman){
//		woman.say();
//	}
	
	public static void say(Person person){
		person.say();
//		person.getAngry(); //由于父类没有该方法，故无法调用
//		Woman woman = (Woman)person; //会报出类型转换异常,因为传入的person可能为man
		
		// 判断person对象是否为woman的实例
		if(person instanceof Woman){
			Woman woman = (Woman)person;
			woman.getAngry();
		}
		
	}
}

abstract class Person{
	private String name;
	public void setName(String name){
		this.name = name;
	}
	
	public String getName(){
		return name;
	}
	
	public abstract void say();//抽象方法
}

class Man extends Person{
	@Override
	public void say() {
		// TODO Auto-generated method stub
		System.out.println("I am a man!!!");
	}
}

class Woman extends Person{
	@Override
	public void say() {
		// TODO Auto-generated method stub
		System.out.println("I am a woman!!!!");
	}
	
	// 本类中进行扩展的方法
	public void getAngry(){
		System.out.println("I am angry!!!");
	}
	
}
```

### 抽象类应用
- 模板方法设计模式
> 定义一个操作中的算法的骨架，而将一些可变部分的实现延迟到子类中。模板方法模式使得子类可以不改变一个算法的结构即可重新定义该算法的某些特定的步骤。


### 接口应用
- 策略设计模式
> 定义了一系列的算法，将每一种算法封装起来并可以相互替换使用，策略模式让算法独立于使用它的客户应用而独立变化。

OO设计原则：
- 面向接口编程（面向抽象编程）
- 封装变化
- 多用组合，少用继承


### Object类
- Object类 是类层次结构的根类
- 每个类都使用Object作为超类。所有对象（包括数组都实现这个类的方法）
- 所有类都是Object类的子类

![这里写图片描述](http://img.blog.csdn.net/20170723133330842?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM5ODM2MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- toString方法
```java
//Object 类的 toString 方法返回一个字符串
//该字符串由类名（对象是该类的一个实例）、at 标记符“@”和此对象哈希码的无符号十六进制表示组成。
public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

- equals()方法
```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```
> equals
> public boolean equals(Object obj)指示其他某个对象是否与此对象“相等”。 
> equals 方法在非空对象引用上实现相等关系： 
>
> - 自反性：对于任何非空引用值 x，x.equals(x) 都应返回 true。 
> - 对称性：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 > true。 
> - 传递性：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。 
> - 一致性：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改。 
对于任何非空引用值 x，x.equals(null) 都应返回 false。 
Object 类的 equals 方法实现对象上差别可能性最大的相等关系；即，对于任何非空引用值 x 和 >y，当且仅当 x 和 y 引用同一个对象时，此方法才返回 true（x == y 具有值 true）。 
>
> 注意：当此方法被重写时，通常有必要重写 hashCode 方法，以维护 hashCode 方法的常规协定，该协定声明相等对象必须具有相等的哈希码。 
>
>
> 参数：
obj - 要与之比较的引用对象。 
返回：
如果此对象与 obj 参数相同，则返回 true；否则返回 false。
另请参见：
hashCode(), Hashtable

- equals 和 == 区别
    - 语义上：==指的是内存引用一样。equals是指的是逻辑相等。逻辑相等具体的意思由编写者决定。
    - 默认情况下(继承自Object类)，equals和==是一样的，除非被覆写(override)了。   
    - 最典型equals已经被override的例子是String； String中的字符串文本相等则视为逻辑相等（s1.equals(s2)==true）。


### 简单工厂模式
> 简单工厂模式是由一个公告对象决定创建出哪一种产品类的实例。简单工厂模式是工厂模式家族中最简单实用的模式。

```java
/**
 * 工厂设计模式：
 * 
 */
public class FactoryDemo {
	
	public static void main(String[] args) {
		
//		ClothDoll clothDoll = new ClothDoll();
//		System.out.println(clothDoll.getInfo());
//		
//		BarbieDoll barbieDoll = new BarbieDoll();
//		System.out.println(barbieDoll.getInfo());
		
		Doll clothDoll = DollFactory.getInstance("ClothDoll");
		if(null != clothDoll)
			System.out.println(clothDoll.getInfo());
		Doll barbieDoll = DollFactory.getInstance("Barbie");
		if(null != barbieDoll)
			System.out.println(barbieDoll.getInfo());
	}
}

//工厂类  降低耦合性（主函数与具体类之间的耦合）  解耦合     转移依赖关系
/**
 * 避免主类直接调用具体类来定义对象，然而被调用类可能发生了一些改变，
 * 那么对于main类的代码将会产生影响，可维护性下降，因为太过于依赖具体的类，
 * 故想让主类不与具体类产生依赖关系，所以选择引入工厂模式，也就是一个中间类，
 * 中间类中根据条件产生相应的对象，依赖具体的类，而主类只是需要调用工程类的
 * 相应方法并传入不同的参数就可以产生相应的对象。
 */
class DollFactory{
	public static Doll getInstance(String name){
		// 根据条件生产不同的对象
		if("ClothDoll".equals(name)){
			return new ClothDoll();
		}else if("Barbie".equals(name)){
			return new BarbieDoll();
		}else{
			return null;
		}
	}
	
}


interface Doll{
	String getInfo();
}

class ClothDoll implements Doll{
	public String getInfo(){
		return "我是布娃娃，我怕脏";
	}
}

class BarbieDoll implements Doll{
	public String getInfo(){
		return "我是芭比娃娃,我美得不可思议";
	}
}
```

### 静态代理模式
> 把很多对象都具有的相同动作抽象出来，创建一个代理类。

```java
/**
 * 代理模式（Proxy）：为其它对象提供一种代理以控制对这个对象的访问
 */
public class ProxyDemo {

	public static void main(String[] args) {
		MiaiPerson person = new MiaiPerson("小白");
		
		//创建代理对象
		Matchmaker matchmaker = new Matchmaker(person);
		matchmaker.miai();
	}
	
}

/**
 * 主题接口
 */
interface Subject{
	public void miai();
}

/**
 * 被代理的类
 */
class MiaiPerson implements Subject{
	private String name;
	public MiaiPerson(String name){
		this.name = name;
	}
	
	public void miai(){
		System.out.println(name+"正在相亲中...");
	}
}

/**
 * 代理类
 */
class Matchmaker implements Subject{
	private Subject target;//要代理的目标对象
	
	public Matchmaker(Subject target){
		this.target = target;
	}
	
	public void before(){
		System.out.println("为代理人匹配如意郎君。");
	}
	
	public void after(){
		System.out.println("本次相亲结束");
	}
	
	@Override
	public void miai() {
		before();
		// 真正执行相亲的方法
		target.miai();
		after();
	}
	
	
}
```

### 适配器模式
> 将一个类的接口转换成客户希望的另一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

> OO设计原则：
> - 面向接口编程
> - 封装变化
> - 多用组合少用继承
> - 对修改关闭，对扩展开放

```java
public class AdapterDemo {
	
	public static void main(String[] args) {
		PowerA powerA = new PowerAImplA();
		startA(powerA);
		
		
		PowerB powerB = new PowerBImpl();
		//startA(powerB); 报错
		PowerAAdapter pAAdapter = new PowerAAdapter(powerB);
		startA(pAAdapter);
		
	}
	
	public static void startA(PowerA powerA){
		// ......
		powerA.insert();
		// ......
	}
	//与startA的内容大部分重复
//	public static void startB(PowerB powerB){
//		// ......
//		powerB.connect();
//		// ......
//	}
}

class PowerAAdapter implements PowerA{
	
	private PowerB powerB;// 要进行适配的接口
	
	public PowerAAdapter(PowerB powerB) {
		this.powerB = powerB;
	}

	@Override
	public void insert() {
		powerB.connect();
	}
	
}

/**
 * 电源A接口
 */
interface PowerA{
	public void insert();
}

class PowerAImplA implements PowerA{

	@Override
	public void insert() {
		// TODO Auto-generated method stub
		System.out.println("电源A接口插入，开始工作。");
	}
	
}

/**
 * 电源B接口
 */
interface PowerB{
	public void connect();
}

class PowerBImpl implements PowerB{
	public void connect(){
		System.out.println("电源B接口已连接。");
	}
}
```
### 内部类
#### 内部类的基本概念
> 在一个类的内部定义的类

```java
class Outer{
    class Inner{
        
    }
}
```
- 编译会产生两个class文件
    - eg.Dog.class
    - eg.Dog$ChildDog.class($符号表示子类)

#### 在外部创建内部类对象
- 内部类除了可以在外部类中产生实例化对象，也可以在外部类的外部来实例化，那么，根据内部类生成的*.class文件：Outer$Inner.class,$符号在程序运行时将替换为“.”，所以内部类的访问可以通过“外部类.内部类”的形式表示

```java
Outer out = new Outer();//产生外部类实例
Outer.Inner in = null;//声明内部类对象
in = out.new Inner();//实例化内部类对象
```

#### 方法内部类
- 内部类除了可以作为一个类的成员以外 ，还可以把类放在方法里定义

```java
class Outer{
    public void doSomething(){
        class Inner{
            public void seeOuter(){
                
            }
        }
    }
}
```


- 示例代码
```java
public class InnerClassDemo1 {
	
	public static void main(String[] args) {
		Dog dog = new Dog("二妞");
		dog.say();
		Dog.ChildDog childDog = null;//声明内部类引用
		
		// 该方式一般不使用
		childDog = dog.new ChildDog();//创建内部类的实例
		childDog.talk();
		
		dog.childTalk();
		
	}

}

class Dog{
	private String name;
	public Dog(String name){
		this.name = name;
	}
	public void say(){
		System.out.println("我是一只狗，主人叫我" + name);
	}
	//内部类(成员内部类)
	class ChildDog{
		public void talk(){
			System.out.println("我是一只小狗狗，我妈是" + name);
		}
	}
	
	public void childTalk(){
		ChildDog childDog = new ChildDog();
		childDog.talk();
	}
}

```
- 注意：
    - 方法内部类只能在定义该内部类的方法内实例化，不可以在此方法外对其实例化。
    - 方法内部类对象不能使用该内部类所在方法的非final局部变量
        - 原因：局部变量位于内存中栈的位置，故方法中的局部变量就是该方法的域，两个大括号之间。而在方法中定义的类的作用域也只能是方法的域，但如果实现外部的接口，则该类创建的对象的作用域将变大，而局部变量的属性只在方法的域中，在外部的时候将无法正常使用。所以需要加final关键字使其变为常量

```java
public class InnerClassDemo2 {

}

class Dog{
	private String name;
	public Dog(String name){
		this.name = name;
	}
	public void say(){
		System.out.println("我是一只狗，主人叫我" + name);
	}
	
	// 在方法里面声明一个内部类
	public void childTalk(final String childName){
		final int x = 10;
		class ChildDog{
			public void talk(){
				System.out.println("我是一只狗狗，我妈妈是"+ name);
				System.out.println("x=" + x);
				System.out.println("我的名字是" + childName);
			}
		}
		ChildDog childDog = new ChildDog();
		childDog.talk();
	}
}
```

#### 静态内部类
- 静态的含义是该内部类可以像其他静态成员一样，没有外部类对象时，也能够访问它。静态嵌套类仅能访问外部类的静态成员和方法。

```java
class Outer{
    static class Inner{}
}
class Test{
    public static void main(String[] args){
        
    }
}
```

#### 匿名内部类
> 匿名内部类就是没有名字的内部类

- 匿名内部类的三种情况
    - 继承式的匿名内部类
    - 接口式的匿名内部类
    - 参数式的匿名内部类

```java
public class InnerClassDemo4 {

	public static void main(String[] args) {
		// 1、继承式匿名内部类
		Dog dog = new Dog("小白"){
			public void say(){
				System.out.println("我是一只womanDog");
			}
		};
		dog.say();
		
		// 2.接口式匿名内部类
		Child child = new Child() {
			
			@Override
			public void talk() {
				// TODO Auto-generated method stub
				System.out.println("我是一只小狗狗");
			}
		};
		child.talk();
	
	
		// 3、参数式匿名内部类
		childTalk(new Child() {
			
			@Override
			public void talk() {
				// TODO Auto-generated method stub
				System.out.println("我是一只小狗狗");
			}
		});
	
	}
	
	
	public static void childTalk(Child child){
		child.talk();
	}
}

class Dog{
	private String name;
	public Dog(String name){
		this.name = name;
	}
	public void say(){
		System.out.println("我是一只狗，主人叫我" + name);
	}
}

class WomanDog extends Dog{
	public WomanDog(String name){
		super(name);
	}
	public void say(){
		System.out.println("我是一只womanDog");
	}
}

interface Child{
	public void talk();
}
```

- 注意：
    - 不能有构造方法，只能有一个实例
    - 不能定义任何静态成员、静态方法
    - 不能是public protected private static 修饰
    - 一定是在new的后面，用其隐含实现一个接口或实现一个类
    - 匿名内部类为局部的，所以局部内部类的所有限制都对其生效。

#### 内部类的作用
- 每个内部类都能独立地继承自一个（接口的）实现，所以无论外部类是否已经继承了某个（接口的）实现，对于内部类没有影响。如果没有内部类提供的可以继承多个具体的或抽象的类的能力，一些设计与编程问题就很难解决。从这个角度看，内部类使得多重继承的解决方案变得完整。接口除了解决了部分问题，而内部类有效地实现了"多重继承"类没有影响。如果没有内部类提供的可以继承多个具体的或抽象的类的能力，一些设计与编程问题就很难解决。从这个角度看，内部类使得多重继承的解决方案变得完整。接口除了解决了部分问题，而内部类有效地实现了"多重继承"