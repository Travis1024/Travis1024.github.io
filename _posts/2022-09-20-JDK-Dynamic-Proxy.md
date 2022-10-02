---
title: JDK Dynamic Proxy
author: Travis <Hongxu Wei>
date: 2022-09-20 20:25:00 +0800
categories: [Java Learning Space]
tags: [动态代理]
math: false
---

## JDK Dynamic Proxy

### 1. 代码结构

#### （1）接口

```java
public interface Service {
    void sing();
    String show(int age);
}
```

#### （2）目标对象（implements接口）（可包含多个）

```java
public class superStarZhou implements Service {
    @Override
    public void sing() {
        System.out.println("我是周mou，我正在唱歌");
    }

    @Override
    public String show(int age) {
        System.out.println("我是周mou，我正在表演------=");
        System.out.println(age);
        return "zhourunfa";
    }
}
```

#### （3）ProxyFactory 代理类

```java
public class ProxyFactory {
    private Service target;

    public ProxyFactory(Service target) {
        this.target = target;
    }
    //拿到动态代理对象
    public Object getAgent(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args)
                            throws Throwable {
                        System.out.println("正在商定日期");
                        System.out.println("正在商量价格");

                        Object obj = method.invoke(target, args);

                        System.out.println("结算费用");
                        return obj;
                    }
                }
        );
    }
}
```

- **代理类可以分为几块：**
  
  1. 构造方法定义接口类，拿到目标对象
     
               private Service target;
  
  2. 创建拿到动态代理对象的方法 
     
               Public Object getAgent()
  
  3. 返回 Proxy.newProxyInstance()方法，其中含有三个参数
     
               ClassLoader loader,
               Class<?>[] interfaces,
               InvocationHandler h
  
  4. 使用匿名内部类，创建InvocationHandler对象，其中重写了invoke()方法
  
  5. invoke()中定义需要额外的代理方法，返回Object obj
     
               Object obj = method.invoke(target, args)

#### （4）测试类

```java
// 创建代理生成类，传入目标对象
ProxyFactory proxyFactory = new ProxyFactory(new SuperStarLiu());
// 通过代理生成类拿到代理对象，并向下转型为接口类对象
Service agent = (Service) proxyFactory.getAgent();
//调用代理类方法，例如
agent.sing();
agent.show(18);
```

### 2. Tips

动态代理类（agent.getClass() == class com.sun.proxy.$Proxy2 ）