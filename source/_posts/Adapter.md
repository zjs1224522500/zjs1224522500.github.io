---
title: '适配器模式'
date: 2017-10-06 20:32:20
tags: 设计模式
---
## 适配器模式

> 将一个类的接口转换成客户希望的另一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

> OO设计原则：
> - 面向接口编程
> - 封装变化
> - 多用组合少用继承
> - 对修改关闭，对扩展开放

<!-- more -->
```java
public class AdapterDemo {
	
	public static void main(String[] args) {
		PowerA powerA = new PowerAImplA();
		startA(powerA);
		
		
		PowerB powerB = new PowerBImpl();
		//startA(powerB); 报错
		//由于 startA 的参数为 PowerA 类型,此时需要适配
		
		PowerAAdapter pAAdapter = new PowerAAdapter(powerB);
		startA(pAAdapter);
		
	}
	
	public static void startA(PowerA powerA){
		// ......
		powerA.insert();
		// ......
	}
	

    //与startA的内容大部分重复，故仅适用 startA() ，从而需要一个相应的适配器，使得 PowerB 能使用 startA()
	public static void startB(PowerB powerB){
		// ......
		powerB.connect();
		// ......
	}
}


//适配器Adapter核心代码
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

![image](http://oxp7d1tae.bkt.clouddn.com/%E9%80%82%E9%85%8D%E5%99%A8%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.png?e=1507795609&token=ZXDhxYzNG4wTZtYl8hbtBHRR-jjSKurOSb6w2aOU:JF_g64CfcHaOEF0Qcsh0PCBTQfw)

