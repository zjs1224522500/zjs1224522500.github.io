---
title: 'tinySpring学习笔记（一）-实现IOC容器'
date: 2017-11-18 16:01:49
tags: Spring
---

## tinySpring学习笔记（一）-实现IOC容器
> 为了更好的理解Spring的核心思想（IOC和AOP），开始阅读并总结体会 [code4craft](https://github.com/code4craft) 的 [tinySpring](https://github.com/code4craft/tiny-spring)模仿Spring的微型项目，并做一些笔记记录。

<!-- more -->
### step1-最基本的容器
> git checkout step-1-container-register-and-get

#### IOC最基本的组成：BeanFactory 和 Bean（容器和实体）
#### 实现步骤：
- 1、初始化beanFactory（容器）
```java
/** 
 * BeanFactory 主要包含一个存储 Bean 的 ConcurrentHashMap 对象，以及对应的注入 Bean 和获取 Bean 的方法
 * Fields:  private Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>();
 * Methods: public Object getBean(String name)
 *          public void registerBeanDefinition(String name, BeanDefinition beanDefinition)
 */
BeanFactory beanFactory = new BeanFactory();
```

- 2、向容器中注入Bean（实体）
```java
/**
 * 为了保存一些额外的元信息，使用 BeanDefinition 来包装对应的 Bean 
 * Fields： private Object bean;
 *          Extra info（Class，ClassName）
 * Method:  public BeanDefinition(Object bean);
 *          属性对应的 set 和 get方法
 */
BeanDefinition beanDefinition = new BeanDefinition(new HelloWorldService());
//注入时，则是利用 HashMap 存放相应的键值对 
beanFactory.registerBeanDefinition("helloWorldService", beanDefinition);
```

- 3、获取对应的Bean实例
```java
// 获取 Bean 时对应的利用 HashMap 从中取出一开始注入的 Bean
HelloWorldService helloWorldService = (HelloWorldService) beanFactory.getBean("helloWorldService");
helloWorldService.helloWorld();
```

### step2-将Bean创建放入工厂
> git checkout step-2-abstract-beanfactory-and-do-bean-initilizing-in-it

- 实际中需要使用容器来对所有的`Bean`进行统一的管理，自然对于`Bean`的整个生命周期都需要容器进行相应的干预。所以对于`Bean`的创建和初始化需要放入容器中进行处理，同时考虑到**程序的扩展性**，需要使用`Extra Interface`将`BeanFactory`更改为接口并提供相应的实现。

- 对于 BeanFactory 的继承关系
```java
/**
 * Interface BeanFactory ->
 * Abstract class AbstractBeanFactory ->
 * class AutowireCapableBeanFactory
 */
public interface BeanFactory {
    Object getBean(String name);
    void registerBeanDefinition(String name, BeanDefinition beanDefinition);
}

public abstract class AbstractBeanFactory implements BeanFactory {
	private Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>();
	@Override
    public Object getBean(String name) {
        //  先获取对应的包装对象，再获取对应的实体
		return beanDefinitionMap.get(name).getBean();
	}
	@Override
    public void registerBeanDefinition(String name, BeanDefinition beanDefinition) {
        // 注入Bean之前先创建对应Bean
        Object bean = doCreateBean(beanDefinition);
        // 创建Bean之后并进行相应的包装
        beanDefinition.setBean(bean);
        // 注入包装后的Bean对象
        beanDefinitionMap.put(name, beanDefinition);
	}
    /**
     * 初始化bean （仅提供抽象方法声明，不提供具体实现）
     * @param beanDefinition
     * @return
     */
    protected abstract Object doCreateBean(BeanDefinition beanDefinition);

}

public class AutowireCapableBeanFactory extends AbstractBeanFactory {
    @Override
    protected Object doCreateBean(BeanDefinition beanDefinition) {
        try {
            // 利用包装类对应的 class 对象，再利用 反射 创建对象
            Object bean = beanDefinition.getBeanClass().newInstance();
            return bean;
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return null;
    }
}

```

#### 实现步骤：（相比step1区别仅在于 ==初始化BeanFactory和注入Bean==）

- 1、初始化BeanFactory
```java
// 创建能够 使用容器创建Bean实例 的容器对象
BeanFactory beanFactory = new AutowireCapableBeanFactory();
```

- 2、注入Bean
```java
/**
 * BeanDefinition:
 * Fields: Object bean;
 *         String beanClassName;
 *         Class beanclass;
 * 其中在 beanClassName 的 set 方法进行注入时，
 * 利用反射和全类名创建出对应的 Class 对象并赋予给 beanClass属性
 * public void setBeanClassName(String beanClassName) {
 *		this.beanClassName = beanClassName;
 *		try {
 *			this.beanClass = Class.forName(beanClassName);
 *		} catch (ClassNotFoundException e) {
 *			e.printStackTrace();
 *		}
 *	}
 */
BeanDefinition beanDefinition = new BeanDefinition();
// 通过使用 包装类BeanDefinition的全类名className 为容器使用反射创建对象做准备
beanDefinition.setBeanClassName("us.codecraft.tinyioc.HelloWorldService");
// 利用反射进行对象的创建并注入
beanFactory.registerBeanDefinition("helloWorldService", beanDefinition);
```

### step3-为Bean注入属性
> git checkout step-3-inject-bean-with-property

- 将属性注入信息保存成`PropertyValue`对象，并且保存到`BeanDefination`，初始化`Bean`的时候，可以根据`PropertyValue`来进行`Bean`属性的注入，`Spring`本身是用了`setter`来注入，该`demo`使用`Field`注入。

#### 实现步骤：
- 1、初始化Factory
```java
BeanFactory beanFactory = new AutowireCapableBeanFactory();
```
- 2、Bean定义
```java
// BeanDefinition 定义中添加对应的 PropertyValues 属性
BeanDefinition beanDefinition = new BeanDefinition();
// 注入全类名便于之后利用反射创建相应的对象
beanDefinition.setBeanClassName("us.codecraft.tinyioc.HelloWorldService");
```
- 3、设置属性
```java
/**
 * PropertyValue: 属性(键值对)
 * Fields： final String name;
 *          final Object value;
 *  
 * PropertyValues：保存对象的全部属性
 * Fields： final List<PropertyValue> propertyValueList;
 * Methods: void addPropertyValue(PropertyValue p);
 */
PropertyValues propertyValues = new PropertyValues();
propertyValues.addPropertyValue(new PropertyValue("text", "Hello World!"));
beanDefinition.setPropertyValues(propertyValues);
```

- 4、生成Bean
```java
/**
 * 1、利用反射创建相应的Bean对象
 * 2、向创建的对象中注入对应的属性（PropertyValues）
 * protected void applyPropertyValues(Object bean, BeanDefinition mbd) throws Exception {
 *      // 遍历 PropertyValues中对应的属性
 *		for (PropertyValue propertyValue : mbd.getPropertyValues().getPropertyValues()) {
 *		    // 获取对象中指定属性名（ PropertyValue 的 key）对应的属性
 *			Field declaredField = bean.getClass().getDeclaredField(propertyValue.getName());
 *			// 设置Access使得私有属性能够通过反射访问
 *			declaredField.setAccessible(true);
 *			// 向实体对象对应的属性字段注入属性值（value） 类似于setter
 *			declaredField.set(bean, propertyValue.getValue());
 *		}
 *	}
 */
beanFactory.registerBeanDefinition("helloWorldService", beanDefinition);
```

- 5、获取 Bean
```java
HelloWorldService helloWorldService = (HelloWorldService) beanFactory.getBean("helloWorldService");
helloWorldService.helloWorld();
```

### step4-读取xml配置来初始化bean
> git checkout step-4-config-beanfactory-with-xml

- 定义`BeanDefinitionReader`接口，提供`void loadBeanDefinitions(String location)`方法从指定位置获取相应的配置来进行初始化，定义抽象类`AbstractBeanDefinitionReader`,其中一个实现是`XmlBeanDefinitionReader`

```java
/**
 * interface BeanDefinitionReader ->
 * abstract class AbstractBeanDefinitionReader ->
 * class XmlBeanDefinitionReader
 */
public interface BeanDefinitionReader {
    void loadBeanDefinitions(String location) throws Exception;
}

public abstract class AbstractBeanDefinitionReader implements BeanDefinitionReader {
    // 保存从配置文件中加载的所有的 beanDefinition 对象
    private Map<String,BeanDefinition> registry;
    /**
     * 依赖 ResourceLoader，该类又依赖 UrlResource 
     * UrlResource 继承自 Spring 自带的 Resource 内部资源定位接口
     * Resource	接口，标识一个外部资源。通过 getInputStream() 方法 获取资源的输入流 。
     * UrlResource 实现 Resource 接口的资源类，通过 URL 获取资源。
     * ResourceLoader 资源加载类。通过 getResource(String) 方法获取一个 Resource 对象，是获取 Resource 的主要途径.
     */
    private ResourceLoader resourceLoader;
    protected AbstractBeanDefinitionReader(ResourceLoader resourceLoader){
        this.registry = new HashMap<String, BeanDefinition>();
        this.resourceLoader = resourceLoader;
    }
    public Map<String, BeanDefinition> getRegistry() {
        return registry;
    }
    public ResourceLoader getResourceLoader() {
        return resourceLoader;
    }
}

public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader{
	public XmlBeanDefinitionReader(ResourceLoader resourceLoader) {
		super(resourceLoader);
	}
    // 从配置文件中加载 Bean 的相关信息
	@Override
	public void loadBeanDefinitions(String location) throws Exception {
		// 利用文件路径创建相应的输入流
		InputStream inputStream = getResourceLoader().getResource(location).getInputStream();
		doLoadBeanDefinitions(inputStream);
	}
	protected void doLoadBeanDefinitions(InputStream inputStream) throws Exception {
		DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
		DocumentBuilder docBuilder = factory.newDocumentBuilder();
		// 输入流转换成对应的 Document 对象便于获取对应的元素
		Document doc = docBuilder.parse(inputStream);
		// 解析bean
		registerBeanDefinitions(doc);
		inputStream.close();
	}
	public void registerBeanDefinitions(Document doc) {
	    // 获取文件中包含的元素 <beans></beans>
		Element root = doc.getDocumentElement();
		parseBeanDefinitions(root);
	}
	protected void parseBeanDefinitions(Element root) {
	    // 获取元素包含的子节点链 <bean></bean> 链
		NodeList nl = root.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
		    // 获取子节点链上对应的子节点 单个<bean></bean>
			Node node = nl.item(i);
			if (node instanceof Element) {
				Element ele = (Element) node;
				// 将节点强转为 Element 对象并进行解析
				processBeanDefinition(ele);
			}
		}
	}
	protected void processBeanDefinition(Element ele) {
	    // 获取子节点对应的属性（name，class）来对应 Bean
	    // <bean name="" class=""></bean>
		String name = ele.getAttribute("name");
		String className = ele.getAttribute("class");
		// 创建与之对应的 BeanDefinition ，并设置相应的属性
        BeanDefinition beanDefinition = new BeanDefinition();
        // 将节点包含的 Bean相关的属性信息注入创建的 BeanDefinition 中。
        processProperty(ele,beanDefinition);
        beanDefinition.setBeanClassName(className);
        // 统一管理（HashMap）通过配置文件加载的 beanDefinition 对象
		getRegistry().put(name, beanDefinition);
	}
    private void processProperty(Element ele,BeanDefinition beanDefinition) {
        // 获取元素对应的 Property 节点
        // <bean><property name="" value=""></property></bean>
        NodeList propertyNode = ele.getElementsByTagName("property");
        for (int i = 0; i < propertyNode.getLength(); i++) {
            // 遍历节点并取出节点对应的 key-value，添加到 BeanDefinition 对应的属性中
            Node node = propertyNode.item(i);
            if (node instanceof Element) {
                Element propertyEle = (Element) node;
                String name = propertyEle.getAttribute("name");
                String value = propertyEle.getAttribute("value");
                beanDefinition.getPropertyValues().addPropertyValue(new PropertyValue(name,value));
            }
        }
    }
}

```

#### 实现步骤：

- 1、读取配置
```java
XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(new ResourceLoader());
xmlBeanDefinitionReader.loadBeanDefinitions("tinyioc.xml");
```

- 2、初始化BeanFactory并注册bean
```java
BeanFactory beanFactory = new AutowireCapableBeanFactory();
// 通过Map自带的Entry实体属性遍历Map中存放的实体集合
for (Map.Entry<String, BeanDefinition> beanDefinitionEntry : xmlBeanDefinitionReader.getRegistry().entrySet()) {
        beanFactory.registerBeanDefinition(beanDefinitionEntry.getKey(), beanDefinitionEntry.getValue());
}
```

- 3、获取bean
```java
HelloWorldService helloWorldService = (HelloWorldService) beanFactory.getBean("helloWorldService");
helloWorldService.helloWorld();
```

### step5-为bean注入bean（处理Bean之间的依赖）
> git checkout step-5-inject-bean-to-bean

- 通过定义一个BeanReference，表示这个属性是对另一个Bean的引用，在读取xml配置文件的时候进行初始化，并在初始化bean的时候，进行解析和真实bean的注入。
```java
class AutowireCapableBeanFactory 中：

protected void applyPropertyValues(Object bean, BeanDefinition mbd) throws Exception {
		for (PropertyValue propertyValue : mbd.getPropertyValues().getPropertyValues()) {
		    // 获取对象中指定属性名（ PropertyValue 的 key）对应的属性
			Field declaredField = bean.getClass().getDeclaredField(propertyValue.getName());
			declaredField.setAccessible(true);
			Object value = propertyValue.getValue();
			// 若该属性的值对应的是一个Bean，是Bean对Bean的引用
			if (value instanceof BeanReference) {
			    // 原类型为 Object，需要强转为 BeanReference
				BeanReference beanReference = (BeanReference) value;
				// 再利用BeanReference存储的name属性获取到对应的bean（包装类 -> 实体类）
				value = getBean(beanReference.getName());
			}
			declaredField.set(bean, value);
		}
	}
	

    /**
     * 对于 getBean(),采用"lazy-init"的方式（延迟加载）。
     * 避免两个循环依赖的Bean在创建时陷入死锁。
     * 在注入Bean的时候，先尝试获取，获取不到再创建，故总是先创建后注入。
     */
    public Object getBean(String name) throws Exception {
		BeanDefinition beanDefinition = beanDefinitionMap.get(name);
		if (beanDefinition == null) {
			throw new IllegalArgumentException("No bean named " + name + " is defined");
		}
		Object bean = beanDefinition.getBean();
		if (bean == null) {
			bean = doCreateBean(beanDefinition);
		}
		return bean;
	}

```

#### 实现步骤：
- 1、读取配置
```java
XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(new ResourceLoader());
xmlBeanDefinitionReader.loadBeanDefinitions("tinyioc.xml");
```

- 2、初始化BeanFactory并注册Bean
```java
AbstractBeanFactory beanFactory = new AutowireCapableBeanFactory();
for (Map.Entry<String, BeanDefinition> beanDefinitionEntry : xmlBeanDefinitionReader.getRegistry().entrySet()) {
    beanFactory.registerBeanDefinition(beanDefinitionEntry.getKey(), beanDefinitionEntry.getValue());
}
```

- 3、初始化Bean
```java
/**
 * class AbstractBeanFactory: 
 * Fileds: + final List<String> beanDefinitionNames 属性
 * Methods：在registerBeanDefinition中对应的添加 beanDefinitionNames.add(name);
 * 
 * public void preInstantiateSingletons() throws Exception {
 *      // 使用迭代器依次创建初始化对应 Bean
 *		for (Iterator it = this.beanDefinitionNames.iterator(); it.hasNext();) {
 *			String beanName = (String) it.next();
 *			getBean(beanName);
 *		}
 *	}
 *
 * 确保所有没有设置lazy-init的单例被实例化
 */
beanFactory.preInstantiateSingletons();
```

- 4、获取Bean
```java
HelloWorldService helloWorldService = (HelloWorldService) beanFactory.getBean("helloWorldService");
helloWorldService.helloWorld();
```


### step6-ApplicationContext
> git checkout step-6-invite-application-context

- 为了便于使用`BeanFactory`，引入`ApplicationContext`接口，并在`AbstractApplicationContext`的`refresh()`方法中进行`bean`的初始化工作。
```java
/**
 * class BeanFactory -> extends
 * interface ApplicationContext -> implements
 * abstract class AbstractApplicationContext -> extends
 * class ClassPathXmlApplicationContext
 */
public interface ApplicationContext extends BeanFactory {
}

public abstract class AbstractApplicationContext implements ApplicationContext {
    // 依赖抽象容器
    protected AbstractBeanFactory beanFactory;
    public AbstractApplicationContext(AbstractBeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }
    //声明方法 初始化所有的 Bean
    public void refresh() throws Exception{
    }
    // 调用容器的获取Bean方法
    @Override
    public Object getBean(String name) throws Exception {
        return beanFactory.getBean(name);
    }
}

public class ClassPathXmlApplicationContext extends AbstractApplicationContext {
	private String configLocation;
	// 默认使用自动装配容器
	public ClassPathXmlApplicationContext(String configLocation) throws Exception {
		this(configLocation, new AutowireCapableBeanFactory());
	}
	// 根据 配置文件路径 容器类型 构造对应的 ApplicationContext
	public ClassPathXmlApplicationContext(String configLocation, AbstractBeanFactory beanFactory) throws Exception {
		super(beanFactory);
		this.configLocation = configLocation;
		refresh();
	}
	
	// 将读取配置文件以及将读取到的属性值注入Bean的方法进行封装
	@Override
	public void refresh() throws Exception {

		// 读取配置文件 封装原来的实现步骤一
		XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(new ResourceLoader());
		xmlBeanDefinitionReader.loadBeanDefinitions(configLocation);
		
		// 将获得的信息注入Bean 封装原来的实现步骤二
		for (Map.Entry<String, BeanDefinition> beanDefinitionEntry : xmlBeanDefinitionReader.getRegistry().entrySet()) {
			beanFactory.registerBeanDefinition(beanDefinitionEntry.getKey(), beanDefinitionEntry.getValue());
		}
	}
}
```
#### 实现步骤：

- 1、初始化ApplicationContext（加载配置文件并初始化 Bean）
```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("tinyioc.xml");
```

- 2、获取对应的 Bean 并执行相关方法
```java
HelloWorldService helloWorldService = (HelloWorldService) applicationContext.getBean("helloWorldService");
helloWorldService.helloWorld();
```

