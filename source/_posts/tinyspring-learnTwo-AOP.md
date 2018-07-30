---
title: 'tinySpring学习笔记（二）-实现AOP'
date: 2018-04-10 16:01:49
tags: Spring
---

## AOP及其实现
> AOP分为配置(Pointcut，Advice)，织入(Weave)两部分工作，当然还有一部分是将AOP整合到整个容器的生命周期中。

### step1-使用JDK动态代理实现AOP织入
> git checkout step-7-method-interceptor-by-jdk-dynamic-proxy

<!-- more -->
- 织入（weave）相对简单，我们先从它开始。Spring AOP的织入点是AopProxy，它包含一个方法Object getProxy()来获取代理后的对象。
```java
public interface AopProxy {
    Object getProxy();
}
```

#### AOP 中两个重要角色：MethodInterceptor和MethodInvocation
- 这两个角色都是AOP联盟的标准，它们分别对应AOP中两个基本角色：`Advice`和`Joinpoint`。`Advice`定义了在切点指定的逻辑，而`Joinpoint`则代表切点。

```java
public interface MethodInterceptor extends Interceptor {
    Object invoke(MethodInvocation invocation) throws Throwable;
}
```
- Spring的AOP只支持方法级别的调用，所以其实在AopProxy里，我们只需要将MethodInterceptor放入对象的方法调用即可。

- 我们称被代理对象为`TargetSource`，而`AdvisedSupport`就是保存`TargetSource`和`MethodInterceptor`的元数据对象。这一步我们先实现一个基于`JDK`动态代理的`JdkDynamicAopProxy`，它可以对接口进行代理。于是我们就有了基本的织入功能。

```java
public class JdkDynamicAopProxy implements AopProxy, InvocationHandler {

    /**
     * AdvisedSupport
     * Fields: TargetSource targetSource
     *         [Object target,Class targetClass];//target 代理对象,targetClass 继承的接口
     *         
     *         MethodInterceptor methodInterceptor;//Advice切面逻辑
     * Methods：对应属性的getter、setter
     */
	private AdvisedSupport advised;

	public JdkDynamicAopProxy(AdvisedSupport advised) {
		this.advised = advised;
	}

    //为要代理的对象的的类对应的创建动态代理
    @Override
	public Object getProxy() {
	
	    /**
	     * public static Object newProxyInstance(ClassLoader loader,
         *                                       Class<?>[] interfaces,
         *                                       InvocationHandler h)
         * @param loader the class loader to define the proxy class
         * @param interfaces the list of interfaces for the proxy class to implement
	     * @param h the invocation handler to dispatch method invocations to
	     */
		return Proxy.newProxyInstance(getClass().getClassLoader(), new Class[] { advised.getTargetSource()
				.getTargetClass() }, this);
	}

    /**
     * 重写 InvocationHandler 对应的 invoke() 方法
     * 调用拦截器对应的方法
     * （通过反射获取对应的切点，再根据切点指定的逻辑进行执行）
     * @Param Object proxy 代理
     * @Param Method method 对应的要调用方法
     * @Param Object[] args 方法需要的参数
     */
	@Override
	public Object invoke(final Object proxy, final Method method, final Object[] args) throws Throwable {
		MethodInterceptor methodInterceptor = advised.getMethodInterceptor();
		
		/**
		 * 其中 class MethodInterceptor extends Interceptor {
		 *           Object invoke(MethodInvocation invocation) throws Throwable;
		 *      }
		 *      class ReflectiveMethodInvocation implements MethodInvocation
		 * 故可通过继承 MethodInterceptor 重写相关 invoke() 方法实现 Advice 逻辑
		 */
		return methodInterceptor.invoke(new ReflectiveMethodInvocation(advised.getTargetSource().getTarget(), method,
				args));
	}
}
```
- ReflectiveMethodInvocation.java
```java
public class ReflectiveMethodInvocation implements MethodInvocation {

	private Object target;

	private Method method;

	private Object[] args;

	public ReflectiveMethodInvocation(Object target, Method method, Object[] args) {
		this.target = target;
		this.method = method;
		this.args = args;
	}

	@Override
	public Method getMethod() {
		return method;
	}

	@Override
	public Object[] getArguments() {
		return args;
	}

    // 反射，相当于 target.method(args);即调用代理对象对应的方法
	@Override
	public Object proceed() throws Throwable {
		return method.invoke(target, args);
	}

	@Override
	public Object getThis() {
		return target;
	}

	@Override
	public AccessibleObject getStaticPart() {
		return method;
	}
}
```

#### 实现步骤：
- 1、设置被代理对象，指定切点（JoinPoint）
```java
// 创建AdvisedSupport对象,设置 Joinpoint
AdvisedSupport advisedSupport = new AdvisedSupport();
TargetSource targetSource = new TargetSource(helloWorldService, HelloWorldService.class);
advisedSupport.setTargetSource(targetSource);
```

- 2、设置拦截器（Advice）
```java
/**
 * // Advice 逻辑实现
 * public class TimerInterceptor implements MethodInterceptor {
 *	@Override
 *	public Object invoke(MethodInvocation invocation) throws Throwable {
 *		long time = System.nanoTime();
 *		System.out.println("Invocation of Method " + invocation.getMethod().getName() + " start!");
 *		Object proceed = invocation.proceed();
 *		System.out.println("Invocation of Method " + invocation.getMethod().getName() + " end! takes " + (System.nanoTime() - time) + " nanoseconds.");
 *		return proceed;
 *	 }
 * }
 */
TimerInterceptor timerInterceptor = new TimerInterceptor();
// 设置Advice
advisedSupport.setMethodInterceptor(timerInterceptor);
```

- 3、创建代理（Proxy）
```java
JdkDynamicAopProxy jdkDynamicAopProxy = new JdkDynamicAopProxy(advisedSupport);
HelloWorldService helloWorldServiceProxy = (HelloWorldService) jdkDynamicAopProxy.getProxy();
```

- 4、基于AOP的调用
```java
helloWorldServiceProxy.helloWorld();
```

### step2-使用AspectJ管理切面
> git checkout step-8-invite-pointcut-and-aspectj

- 完成了织入之后，我们要考虑另外一个问题：对什么类以及什么方法进行`AOP`？对于“在哪切”这一问题的定义，我们又叫做“`Pointcut`”。`Spring`中关于`Pointcut`包含两个角色：`ClassFilter`和`MethodMatcher`，分别是对类和方法做匹配。`Pointcut`有很多种定义方法，例如类名匹配、正则匹配等，但是应用比较广泛的应该是和`AspectJ`表达式的方式。
```java
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();

}


public interface ClassFilter {

    boolean matches(Class targetClass);
}

public interface MethodMatcher {

    boolean matches(Method method, Class targetClass);
}
```

- `AspectJ`是一个“对`Java`的`AOP`增强”。它最早是其实是一门语言，我们跟写`Java`代码一样写它，然后静态编译之后，就有了`AOP`的功能。下面是一段`AspectJ`代码：
```java
aspect PointObserving {
    private Vector Point.observers = new Vector();
    public static void addObserver(Point p, Screen s) {
        p.observers.add(s);
    }
    public static void removeObserver(Point p, Screen s) {
        p.observers.remove(s);
    }
    ...
}
```

- 这种方式无疑太重了，为了AOP，还要适应一种语言？所以现在使用也不多，但是它的Pointcut表达式被Spring借鉴了过来。于是我们实现了一个AspectJExpressionPointcut：
```java
public class AspectJExpressionPointcut implements Pointcut, ClassFilter, MethodMatcher {

    // 对用户自定义的相关关键字子集合，可进行切点表达式PointcutExpression的构造
	private PointcutParser pointcutParser;

    // 表达式字符串
	private String expression;

    // 要构造的切点表达式
	private PointcutExpression pointcutExpression;

    // 存储表达式相关关键字
	private static final Set<PointcutPrimitive> DEFAULT_SUPPORTED_PRIMITIVES = new HashSet<PointcutPrimitive>();

	static {
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.EXECUTION);
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.ARGS);
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.REFERENCE);
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.THIS);
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.TARGET);
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.WITHIN);
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.AT_ANNOTATION);
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.AT_WITHIN);
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.AT_ARGS);
		DEFAULT_SUPPORTED_PRIMITIVES.add(PointcutPrimitive.AT_TARGET);
	}
    
    // 进行关键字集合初始化
	public AspectJExpressionPointcut() {
		this(DEFAULT_SUPPORTED_PRIMITIVES);
	}

    // 初始化表达式构造器
	public AspectJExpressionPointcut(Set<PointcutPrimitive> supportedPrimitives) {
		pointcutParser = PointcutParser
				.getPointcutParserSupportingSpecifiedPrimitivesAndUsingContextClassloaderForResolution(supportedPrimitives);
	}

	protected void checkReadyToMatch() {
		if (pointcutExpression == null) {
			pointcutExpression = buildPointcutExpression();
		}
	}

    // 字符串转换为切点表达式
	private PointcutExpression buildPointcutExpression() {
		return pointcutParser.parsePointcutExpression(expression);
	}

	public void setExpression(String expression) {
		this.expression = expression;
	}

    // 对类做匹配
	@Override
	public ClassFilter getClassFilter() {
		return this;
	}

    // 对方法做匹配
	@Override
	public MethodMatcher getMethodMatcher() {
		return this;
	}

    // 将表达式和类做匹配，返回匹配结果
	@Override
	public boolean matches(Class targetClass) {
		checkReadyToMatch();
		return pointcutExpression.couldMatchJoinPointsInType(targetClass);
	}

    // 将表达式和方法做匹配，返回匹配结果
	@Override
	public boolean matches(Method method, Class targetClass) {
		checkReadyToMatch();
		ShadowMatch shadowMatch = pointcutExpression.matchesMethodExecution(method);
		if (shadowMatch.alwaysMatches()) {
			return true;
		} else if (shadowMatch.neverMatches()) {
			return false;
		}
		// TODO:其他情况不判断了！见org.springframework.aop.aspectj.RuntimeTestWalker
		return false;
	}
}
```

#### 实现测试：
```java
// 对类做匹配 返回匹配结果
@Test
public void testClassFilter() throws Exception {
    String expression = "execution(* us.codecraft.tinyioc.*.*(..))";
    AspectJExpressionPointcut aspectJExpressionPointcut = new AspectJExpressionPointcut();
    aspectJExpressionPointcut.setExpression(expression);
    boolean matches = aspectJExpressionPointcut.getClassFilter().matches(HelloWorldService.class);
    Assert.assertTrue(matches);
}

// 对方法做匹配 返回匹配结果
@Test
public void testMethodInterceptor() throws Exception {
    String expression = "execution(* us.codecraft.tinyioc.*.*(..))";
    AspectJExpressionPointcut aspectJExpressionPointcut = new AspectJExpressionPointcut();
    aspectJExpressionPointcut.setExpression(expression);
    boolean matches = aspectJExpressionPointcut.getMethodMatcher().matches(HelloWorldServiceImpl.class.getDeclaredMethod("helloWorld"),HelloWorldServiceImpl.class);
    Assert.assertTrue(matches);
}
```


### step3-将AOP融入Bean的创建过程中
> git checkout step-9-auto-create-aop-proxy

- 在step1 中已经能够进行 weave 织入，step2 中实现了 Pointcut 的匹配。现在需要在 Spring 中整合这两者。Spring给了一个巧妙的答案：使用 `BeanPostProcessor`。
- `BeanPostProcessor`是`BeanFactory`提供的，在`Bean`初始化过程中进行扩展的接口。只要你的`Bean`实现了`BeanPostProcessor`接口，那么Spring在初始化时，会优先找到它们，并且在`Bean`的初始化过程中，调用这个接口，从而实现对`BeanFactory`核心无侵入的扩展。
- `AOP`的xml配置
```xml
<aop:aspectj-autoproxy/>

<!-- 等价 -->
<bean id="autoProxyCreator" class="org.springframework.aop.aspectj.autoproxy.AspectJAwareAdvisorAutoProxyCreator"></bean>
```

- `AspectJAwareAdvisorAutoProxyCreator`就是`AspectJ`方式实现织入的核心。它其实是一个`BeanPostProcessor`。在这里它会扫描所有`Pointcut`，并对`bean`做织入。
- BeanPostProcessor:
```java
public interface BeanPostProcessor {

	Object postProcessBeforeInitialization(Object bean, String beanName) throws Exception;

	Object postProcessAfterInitialization(Object bean, String beanName) throws Exception;

}
```
- AspectJAwareAdvisorAutoProxyCreator:
```Java
public class AspectJAwareAdvisorAutoProxyCreator implements BeanPostProcessor, BeanFactoryAware {

	private AbstractBeanFactory beanFactory;

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws Exception {
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws Exception {
		if (bean instanceof AspectJExpressionPointcutAdvisor) {
			return bean;
		}
        if (bean instanceof MethodInterceptor) {
            return bean;
        }
        
        /**
         * class AspectJExpressionPointcutAdvisor implements PointcutAdvisor
         * Fields: AspectJExpressionPointcut pointcut
         *         Advice advice
         * Methods: Advice getter/setter
         *          Pointcut getter
         *          void setExpression(String expression) {this.pointcut.setExpression(expression);}
         * 
         */
        // 根据 Type 获取所有的 PointCut 和 Advice 组成的 bean
		List<AspectJExpressionPointcutAdvisor> advisors = beanFactory
				.getBeansForType(AspectJExpressionPointcutAdvisor.class);
		for (AspectJExpressionPointcutAdvisor advisor : advisors) {
		    
		    // 判断是否是要拦截的类
			if (advisor.getPointcut().getClassFilter().matches(bean.getClass())) {
				
				/**
				 * Class AdvisedSupport
				 * Fields: TargetSource targetSource;
				 *         MethodInterceptor methodInterceptor;
				 *         MethodMatcher methodMatcher;
				 * Methods: getter/setter
				 */
				AdvisedSupport advisedSupport = new AdvisedSupport();
				
				// 从扫描出的 bean 中获取 Advice 逻辑并注入
				// 设置 Advice
				advisedSupport.setMethodInterceptor((MethodInterceptor) advisor.getAdvice());
				
				/**
				 * MethodMatcher getMethodMatcher() {return this;}
				 * AspectJExpressionPointcut 实现了 MethodMatcher 接口。
				 */
				// 设置切点 Pointcut,即哪些方法需要做拦截
				advisedSupport.setMethodMatcher(advisor.getPointcut().getMethodMatcher());

                // 设置要代理的对象
				TargetSource targetSource = new TargetSource(bean, bean.getClass().getInterfaces());
				advisedSupport.setTargetSource(targetSource);

                // 创建动态代理
				return new JdkDynamicAopProxy(advisedSupport).getProxy();
			}
		}
		return bean;
	}

    // 获取容器的引用，进而获取容器中所有的切点对象
	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws Exception {
		this.beanFactory = (AbstractBeanFactory) beanFactory;
	}
}

```

- 此时的`JdkDynamicAopProxy`
```java
public class JdkDynamicAopProxy implements AopProxy, InvocationHandler {

	private AdvisedSupport advised;

	public JdkDynamicAopProxy(AdvisedSupport advised) {
		this.advised = advised;
	}

	@Override
	public Object getProxy() {
		return Proxy.newProxyInstance(getClass().getClassLoader(), advised.getTargetSource().getTargetClass(), this);
	}

	@Override
	public Object invoke(final Object proxy, final Method method, final Object[] args) throws Throwable {
		MethodInterceptor methodInterceptor = advised.getMethodInterceptor();
	    
	    // invoke 时判断是否为要拦截的方法，是则执行 Advice 逻辑
		if (advised.getMethodMatcher() != null
				&& advised.getMethodMatcher().matches(method, advised.getTargetSource().getTarget().getClass())) {
			return methodInterceptor.invoke(new ReflectiveMethodInvocation(advised.getTargetSource().getTarget(),
					method, args));
		} else {
			return method.invoke(advised.getTargetSource().getTarget(), args);
		}
	}
}
```

- 为了简化`xml`配置，在tiny-spring中直接使用Bean的方式，而不是用aop前缀进行配置：
```xml
<bean id="autoProxyCreator" class="us.codecraft.tinyioc.aop.AspectJAwareAdvisorAutoProxyCreator"></bean>

<bean id="timeInterceptor" class="us.codecraft.tinyioc.aop.TimerInterceptor"></bean>

<!-- Creator 将对 Advisor 类型的 bean 进行扫描和处理 -->
<bean id="aspectjAspect" class="us.codecraft.tinyioc.aop.AspectJExpressionPointcutAdvisor">
    <!-- 调用对应的 setter 方法进行 Property 的注入 -->
    <property name="advice" ref="timeInterceptor"></property>
    <property name="expression" value="execution(* us.codecraft.tinyioc.*.*(..))"></property>
</bean>
```

##### 动态代理的步骤
- `AutoProxyCreator`（实现了 `BeanPostProcessor` 接口）在实例化所有的 `Bean` 前，最先被实例化。`postProcessBeforeInitialization`
- 其他普通 `Bean` 被实例化、初始化，在初始化的过程中，`AutoProxyCreator` 加载 `BeanFactory` 中所有的 `PointcutAdvisor`（这也保证了 `PointcutAdvisor` 的实例化顺序优于普通 `Bean`。），然后依次使用 `PointcutAdvisor` 内置的 `ClassFilter`，判断当前对象是不是要拦截的类。
- 如果是，则生成一个 `TargetSource`（要拦截的对象和其类型），并取出 `AutoProxyCreator` 的 `MethodMatcher`（对哪些方法进行拦截）、`Advice`（拦截的具体操作），再交给 `AopProxy` 去生成代理对象。
- `AopProxy` 生成一个 `InvocationHandler`，在它的 `invoke` 函数中，首先使用 `MethodMatcher` 判断是不是要拦截的方法，如果是则交给 `Advice` 来执行（`Advice` 由用户来编写，其中也要手动/自动调用原始对象的方法），如果不是，则直接交给 `TargetSource` 的原始对象来执行。


### step4-使用CGLib进行类的织入
> git checkout step-10-invite-cglib-and-aopproxy-factory

- 前面的`JDK`动态代理只能对接口进行代理，对于类则无能为力。这里我们需要一些字节码操作技术。这方面大概有几种选择：`ASM`，`CGLib`和`javassist`，后两者是对`ASM`的封装。`Spring`中使用了`CGLib`。
- 在这一步，我们还要定义一个工厂类`ProxyFactory`，用于根据`TargetSource`类型自动创建代理，这样就需要在调用者代码中去进行判断。
- TargetSource:
```java
public class TargetSource {

	private Class<?> targetClass;

    private Class<?>[] interfaces;

	private Object target;

	public TargetSource(Object target, Class<?> targetClass,Class<?>... interfaces) {
		this.target = target;
		this.targetClass = targetClass;
        this.interfaces = interfaces;
	}

	public Class<?> getTargetClass() {
		return targetClass;
	}

	public Object getTarget() {
		return target;
	}

    public Class<?>[] getInterfaces() {
        return interfaces;
    }
}
```
- ProxyFactory : 策略模式和工厂模式的结合使用
```java
public class ProxyFactory extends AdvisedSupport implements AopProxy {

    // 在 creator 中，生成相应的动态代理的时候就可以使用工厂类的 getProxy()
	@Override
	public Object getProxy() {
		return createAopProxy().getProxy();
	}

	protected final AopProxy createAopProxy() {
		return new Cglib2AopProxy(this);
	}
	
	//...可以根据 TargetSource 决定使用不同的动态代理
	protected final AopProxy createAopProxy(TargetSource targetSource) {
	    return new JdkDynamicAopProxy(this);
	}
}
```

- Cglib2AopProxy
```Java
public class Cglib2AopProxy extends AbstractAopProxy {

	public Cglib2AopProxy(AdvisedSupport advised) {
		super(advised);
	}

	@Override
	public Object getProxy() {
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(advised.getTargetSource().getTargetClass());
		enhancer.setInterfaces(advised.getTargetSource().getInterfaces());
		enhancer.setCallback(new DynamicAdvisedInterceptor(advised));
		Object enhanced = enhancer.create();
		return enhanced;
	}

	private static class DynamicAdvisedInterceptor implements MethodInterceptor {

		private AdvisedSupport advised;

		private org.aopalliance.intercept.MethodInterceptor delegateMethodInterceptor;

		private DynamicAdvisedInterceptor(AdvisedSupport advised) {
			this.advised = advised;
			this.delegateMethodInterceptor = advised.getMethodInterceptor();
		}

		@Override
		public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
			if (advised.getMethodMatcher() == null
					|| advised.getMethodMatcher().matches(method, advised.getTargetSource().getTargetClass())) {
				return delegateMethodInterceptor.invoke(new CglibMethodInvocation(advised.getTargetSource().getTarget(), method, args, proxy));
			}
			return new CglibMethodInvocation(advised.getTargetSource().getTarget(), method, args, proxy).proceed();
		}
	}

	private static class CglibMethodInvocation extends ReflectiveMethodInvocation {

		private final MethodProxy methodProxy;

		public CglibMethodInvocation(Object target, Method method, Object[] args, MethodProxy methodProxy) {
			super(target, method, args);
			this.methodProxy = methodProxy;
		}

		@Override
		public Object proceed() throws Throwable {
			return this.methodProxy.invoke(this.target, this.arguments);
		}
	}

}
```
- 代理模式相关参见 [代理模式（Proxy Pattern）- 最易懂的设计模式解析](https://blog.csdn.net/carson_ho/article/details/54910472)