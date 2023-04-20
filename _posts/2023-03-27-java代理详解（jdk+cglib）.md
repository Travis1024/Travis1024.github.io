---
title: java代理详解（jdk+cglib）
author: Travis <Hongxu Wei>
date: 2023-03-27 11:41:44 +0800
categories: [Java Learning Space]
tags: [动态代理]
math: false
---



> RPC中很重要的一点就是动态代理，做个记录



## 一、代理模式

代理模式是常见的设计模式之一，Java我们通常通过new一个对象然后调用其对应的方法来访问我们需要的服务。代理模式则是通过创建代理类（proxy）的方式来访问服务，代理类通常会持有一个委托类对象，代理类不会自己实现真正服务，而是通过调用委托类对象的相关方法，来提供服务，所以其实我们调用的还是委托类的服务。

但是中间隔了一个代理类。这么做是有好处的，我们可以在访问服务之前或者之后加一下我们需要的操作。例如Spring的面向切面编程，我们可以在切入点之前执行一些操作，切入点之后执行一些操作。这个切入点就是一个个方法。这些方法所在类肯定就是被代理了，在代理过程中切入了一些其他操作。

<img src="https://travisnotes.oss-cn-shanghai.aliyuncs.com/mdpic/202303271417188.png" alt="image-20230327141758167" style="zoom:40%;" />



## 二、静态代理

再说动态代理之前先说一下静态代理。静态代理就是程序员在编写代码的时候就已经把代理类的源码写好了，编译后就会生成.class文件。

ℹ️	**静态代理简单实现**

静态代理在使用时,需要定义接口或者父类,被代理对象与代理对象一起实现相同的接口或者是继承相同父类。我这举一个租客通过中介租房子的例子。

- 首先创建一个person接口，这个接口就是租客和中介的公共接口，这个接口有一个rentHouse()方法。

  ```java
  public interface Person {
  	//租房
  	public void rentHouse();
  }
  ```

- 创建租客Renter类，实现上述接口

  ```java
  public class Renter implements Person{
  	@Override
  	public void rentHouse() {
  		System.out.println("租客租房成功！");
  	}
  }
  ```

- 创建中介类RenterProxy，同样实现Person接口，但是还另外持有一个租客类对象

  ```java
  public class RenterProxy implements Person{
  	private Person renter;
  	public RenterProxy(Person renter){
  		this.renter = renter;
  	}
  	@Override
  	public void rentHouse() {
  		System.out.println("中介找房东租房，转租给租客！");
  		renter.rentHouse();
  		System.out.println("中介给租客钥匙，租客入住！");
  	}
  }
  ```

- 新建测试类测试

  ```java
  public class StaticProxyTest {
  	public static void main(String[] args) {
  		Person renter = new Renter();
  		RenterProxy proxy = new RenterProxy(renter);
  		proxy.rentHouse();
  	}
  }
  ```

- 运行结果：

  ```
  中介找房东租房，转租给租客！
  租客租房成功！
  中介给租客钥匙，租客入住！
  ```

  

## 三、动态代理

代理类在程序运行时创建的代理方式被成为动态代理。在静态代理中，代理类（RenterProxy）是自己已经定义好了的，在程序运行之前就已经编译完成。

而动态代理是在运行时根据我们在Java代码中的“指示”动态生成的。动态代理相较于静态代理的优势在于可以很方便的对代理类的所有函数进行统一管理，如果我们想在每个代理方法前都加一个方法，如果代理方法很多，我们需要在每个代理方法都要写一遍，很麻烦。而动态代理则不需要。

ℹ️	**动态代理简单实现**

在java的java.lang.reflect包下提供了一个Proxy类和一个InvocationHandler接口，通过这个类和这个接口可以生成JDK动态代理类和动态代理对象。

- 定义一个person接口

  ```java
  public interface Person {
  	//租房
  	public void rentHouse();
  }
  ```

- 创建被代理的类

  ```java
  public class Renter implements Person{
  	@Override
  	public void rentHouse() {
  		System.out.println("租客租房成功！");
  	}
  }
  ```

- 创建InvocationHandler类，这个类实现了InvocationHandler接口，并持有一个被代理类的对象，InvocationHandler中有一个invoke方法，所有执行代理对象的方法都会被替换成执行invoke方法。然后通过反射在invoke方法中执行代理类的方法。在代理过程中，在执行代理类的方法前或者后可以执行自己的操作，这就是spring AOP的主要原理。

  ```java
  public class RenterInvocationHandler<T> implements InvocationHandler{
  	//被代理类的对象
  	private T target;
  	
  	public RenterInvocationHandler(T target){
  		this.target = target;
  	}
  	/**
       * proxy:代表动态代理对象
       * method：代表正在执行的方法
       * args：代表调用目标方法时传入的实参
       */
  	@Override
  	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  		//代理过程中插入其他操作
  		System.out.println("租客和中介交流");
  		Object result = method.invoke(target, args);
  		return result;
  	}
  }
  ```

- 创建动态代理对象

  ```java
  public class ProxyTest {
  
  	public static void main(String[] args) {
  
  		//创建被代理的实例对象
  		Person renter = new Renter();
  		//创建InvocationHandler对象
  		InvocationHandler renterHandler = new RenterInvocationHandler<Person>(renter);
  		
  		//创建代理对象,代理对象的每个执行方法都会替换执行Invocation中的invoke方法
  		Person renterProxy = (Person)Proxy.newProxyInstance(Person.class.getClassLoader(),new Class<?>[]{Person.class}, renterHandler);
  		renterProxy.rentHouse();
  		
  		//也可以使用下面的方式创建代理类对象，Proxy.newProxyInstance其实就是对下面代码的封装
  		/*try {
  			//使用Proxy类的getProxyClass静态方法生成一个动态代理类renterProxy 
  			Class<?> renterProxyClass = Proxy.getProxyClass(Person.class.getClassLoader(), new Class<?>[]{Person.class});
  			//获取代理类renterProxy的构造器，参数为InvocationHandler
  			Constructor<?> constructor = renterProxyClass.getConstructor(InvocationHandler.class);
  			//使用构造器创建一个代理类实例对象
  			Person renterProxy = (Person)constructor.newInstance(renterHandler);
  			renterProxy.rentHouse();
  			//
  		} catch (Exception e) {
  			// TODO Auto-generated catch block
  			e.printStackTrace();
  		}*/
  	}
  }
  ```

- 执行结果

  ```
  租客和中介交流
  租客租房成功！
  ```

  

ℹ️	**原理分析**

- newProxyInstance源码

  ```
   public static Object newProxyInstance(ClassLoader loader,
                                            Class<?>[] interfaces,
                                            InvocationHandler h)
          throws IllegalArgumentException {
          if (h == null) {
              throw new NullPointerException();
          }
  
          final SecurityManager sm = System.getSecurityManager();
          if (sm != null) {
              checkProxyAccess(Reflection.getCallerClass(), loader, interfaces);
          }
  
          /*
           * Look up or generate the designated proxy class.
           */
           //生成动态代理类
          Class<?> cl = getProxyClass0(loader, interfaces);
  
          /*
           * Invoke its constructor with the designated invocation handler.
           */
          try {
              //获取构造器参数是InvocationHandler类，constructorParams是Proxy的静态成员变量
              // private final static Class[] constructorParams ={ InvocationHandler.class };
              final Constructor<?> cons = cl.getConstructor(constructorParams);
              final InvocationHandler ih = h;
              if (sm != null && ProxyAccessHelper.needsNewInstanceCheck(cl)) {
                  // create proxy instance with doPrivilege as the proxy class may
                  // implement non-public interfaces that requires a special permission
                  return AccessController.doPrivileged(new PrivilegedAction<Object>() {
                      public Object run() {
                          return newInstance(cons, ih);
                      }
                  });
              } else {
                  return newInstance(cons, ih);
              }
          } catch (NoSuchMethodException e) {
              throw new InternalError(e.toString());
          }
      }
  
      private static Object newInstance(Constructor<?> cons, InvocationHandler h) {
          try {
             //使用构造器创建一个代理类实例对象
              return cons.newInstance(new Object[] {h} );
          } catch (IllegalAccessException | InstantiationException e) {
              throw new InternalError(e.toString());
          } catch (InvocationTargetException e) {
              Throwable t = e.getCause();
              if (t instanceof RuntimeException) {
                  throw (RuntimeException) t;
              } else {
                  throw new InternalError(t.toString());
              }
          }
      }
  ```

  Class<?> cl = getProxyClass0(loader, interfaces);这行代码生成了一个代理类，这个类缓存在java虚拟机中。可以通过下列代码将其打印出来，将class文件进行反编译

  ```java
  public final class $Proxy0 extends Proxy implements Person {
    private static Method m1;
    private static Method m3;
    private static Method m0;
    private static Method m2;
    //通过带InvocationHandler参数的构造器生成代理类对象时，将我们写的RenterInvocationHandler对象传进来
    //父类持有：protected InvocationHandler h
    public $Proxy0(InvocationHandler paramInvocationHandler) throws {
      super(paramInvocationHandler);
    }
    
    public final boolean equals(Object paramObject) throws {
      try {
        return ((Boolean)this.h.invoke(this, m1, new Object[] { paramObject })).booleanValue();
      }
      catch (Error|RuntimeException localError) {
        throw localError;
      }
      catch (Throwable localThrowable) {
        throw new UndeclaredThrowableException(localThrowable);
      }
    }
  
    public final void rentHouse() throws {
      try {
        //执行代理类的rentHouse方法时，实际上是执行InvocationHandler类的invoke方法，也就是我们自己写的
        //RenterInvocationHandler类的invoke方法，在此方法中我们通过method.invoke(target, args)反射，再
        //执行具体类的rentHouse方法      
        this.h.invoke(this, m3, null);
        return;
      }
      catch (Error|RuntimeException localError)
      {
        throw localError;
      }
      catch (Throwable localThrowable)
      {
        throw new UndeclaredThrowableException(localThrowable);
      }
    }
  
    public final int hashCode() throws {
      try {
        return ((Integer)this.h.invoke(this, m0, null)).intValue();
      }
      catch (Error|RuntimeException localError) {
        throw localError;
      }
      catch (Throwable localThrowable) {
        throw new UndeclaredThrowableException(localThrowable);
      }
    }
  
    public final String toString() throws {
      try {
        return (String)this.h.invoke(this, m2, null);
      }
      catch (Error|RuntimeException localError) {
        throw localError;
      }
      catch (Throwable localThrowable) {
        throw new UndeclaredThrowableException(localThrowable);
      }
    }
  
    static {
      try {
        m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { 
        Class.forName("java.lang.Object") });
        //获取到了rentHouse方法的方法名
        m3 = Class.forName("test.father.Person").getMethod("rentHouse", new Class[0]);
        m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
        m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
        return;
      }
      catch (NoSuchMethodException localNoSuchMethodException) {
        throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
      }
      catch (ClassNotFoundException localClassNotFoundException) {
        throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
      }
    }
  }
  ```



java自动生成了一个$Proxy0代理类，这个类在内存中，所以可以通过反射获取这个类的构造方法，然后创建的代理类实例对象。分析这个类源码，可以知道当调用代理类对象的rentHouse方法时的大概流程为：调用RenterInvocationHandler类的invoke方法，而RenterInvocationHandler类的invoke方法中又用反射调用了被代理类的rentHouse方法。
RenterInvocationHandler可以看成是中间类，它持有被代理对象，把外部对invoke的调用转为对被代理对象的调用。而代理类通过持有中间类，调用中间类的invoke方法，来达到调用被代理类的方法的目的。

## 四、cglib代理

动态代理需要被代理类实现接口，如果被代理类没有实现接口，那么这么实现动态代理？这时候就需要用到CGLib了。这种代理方式就叫做CGlib代理。

Cglib代理也叫作子类代理，他是通过在内存中构建一个子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，然后加入自己需要的操作。⚠️**因为使用的是继承的方式，所以不能代理final 类**。

ℹ️	cglib代理简单实现

- 创建被代理类HelloImpl

  ```java
  public class HelloImpl{
      public void sayHello() {
          // 你要写的业务代码
          System.out.println("Hello World");
      }
  }
  ```

- 创建代理工厂类

  ```java
  public class MyMethodInterceptor implements MethodInterceptor { 
      private Object target;
      public MyMethodInterceptor(Object target) { //暂时存放target
          this.target = target;
      }
   
      /**
       * 实现MethodInterceptor中的intercept方法
       * 参数 o: cglib 动态生成的代理类的实例
       * method:实体类所调用的都被代理的方法的引用
       * objects 参数列表
       * methodProxy:生成的代理类对方法的代理引用
       **/
      @Override
      public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
          System.out.println("Invoking sayHello");
          Object object=  methodProxy.invoke(target,objects);
          System.out.println("Destory sayHello");
          return object;
      }
  }
  ```

- 实现MethodInterceptor接口，写你要做的代理操作

  ```java
  public class AA {
      public static void main(String[] args) {
          //创建被代理对象
          HelloImpl helloImpl = new HelloImpl();
          //创建代理类
          MyMethodInterceptor interceptor = new MyMethodInterceptor(helloImpl);
          //可以通过Enhancer对象中的create()方法可以去生成一个类，用于生成代理对象
          Enhancer enhancer=new Enhancer();
          //设置父类(将被代理类作为代理类的父类)
          enhancer.setSuperclass(helloImpl.getClass());
        	//设置拦截器(代理类中的intercept就是)
          enhancer.setCallback(interceptor);
   
        	//生成一个被代理类的子类对象
          HelloImpl proxyHelloImpl = (HelloImpl) enhancer.create();
          
          // 调用代理方法
          proxyHello.sayHello();
      }
  }
  ```

  

  动态代理与CGLib动态代理都是实现Spring AOP的基础。**如果加入容器的目标对象有实现接口,用动态代理，如果目标对象没有实现接口，用Cglib代理。**



## 五、JDK和cglib实现动态代理各自优势

- JDK和cglib实现动态代理的区别
  - JDK实现动态代理主要是以接口为中心。JDK提供的代理类，会实现被代理对象的接口。
  - cglib主要原理是为被代理类生成子类，通过继承被代理对象的方法去实现动态代理
  - 如果对象有接口实现，选择JDK代理，如果没有接口实现选择CGILB代理
- JDK和cglib实现动态代理的各自优势
  - JDK：
    - 最小化依赖关系，减少依赖意味着简化开发和维护，JDK本身的支持，可能比cglib更加可靠。
    - 平滑进行JDK版本升级，而字节码类库通常需要进行更新以保证在新版Java上能够使用。
    - 代码实现简单。
  - cglib：
    - 有的时候调用目标可能不便实现额外接口，从某种角度看，限定调用者实现接口是有些侵入性的实践，类似cglib动态代理就没有这种限制。
    - 只操作我们关心的类，而不必为其他相关类增加工作量。
    - cglib性能高于JDK。
    - 被final修饰的方法，无法通过cglib实现动态代理



## 附录

[java动态代理](https://blog.csdn.net/qq_34609889/article/details/85317582)

[java动态代理知乎](https://www.zhihu.com/question/20794107/answer/658139129)