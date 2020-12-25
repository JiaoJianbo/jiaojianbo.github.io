---
layout: post
title: "Spring AOP 和 Java 动态代理的对照说明"
date: 2019-12-25 10:30:00+0800
categories: Spring
tags: Spring
author: Bobby
---

* content
{:toc}

众所周知，Spring AOP 就是基于动态代理实现的。只不过有些情况下使用 JDK 中原生的动态代理，有些使用 Cglib 实现。至于这其中的差异和实现细节这里不做过多的深究，本文仅从宏观的角度对两者的实现代码作一比较，加深对两者之间关系的认识。



### 一个 Java 动态代理类的实现

以下是一个使用 JDK 中的 Proxy 实现的一个打印日志代理类。

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

/**
 * @author Bobby
 */
public class ArithmeticServiceLoggingProxy { //不用实现接口
    // 被代理的目标对象
    private ArithmeticService target;
    
    public ArithmeticServiceLoggingProxy(ArithmeticService target) {
        super();
        this.target = target;
    }

    public ArithmeticService getLoggingProxy() {
        //代理对象由哪一个 ClassLoader 负责加载
        ClassLoader loader = target.getClass().getClassLoader();
        
        //代理对象的类型，实现了哪些接口
        Class<?>[] interfaces = new Class[] {ArithmeticService.class};
        
        //当调用代理对象的方法时，要执行的代码
        /*
         * 主要逻辑其实都在 InvocationHandler 里面，如果可以单出抽出去，那么这里的逻辑可以应用到任何 target 上
         * 参见 LoggingInterceptor
         */
        InvocationHandler h = new InvocationHandler() {
            /**
             * proxy : 正在返回的那个对象，一般情况下，在 invoke 方法中不使用该对象
             * method: 正在被调用的方法
             * args  : 调用方法时，传入的参数
             */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //System.out.println(proxy.toString()); //java.lang.StackOverflowError
                String methodName = method.getName();
                // 前面日志
                System.out.println("Method " + methodName + " begins with " + Arrays.asList(args));
                Object result = method.invoke(target, args);
                // 后面日志
                System.out.println("Method " + methodName + " ends with: " + result);
                return result;
            }
        };

        ArithmeticService proxy = (ArithmeticService) Proxy.newProxyInstance(loader, interfaces, h);
        
        return proxy;
    }
    
}
```

可以将上面 InvocationHandler 的实现类单独抽象出来，这样这个日志功能可以被应用到更多的代理目标对象上。

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.util.Arrays;

/**
 * 单独抽出日功能，这样该功能就可以被应用到任何被代理的对象上
 * 
 * @author Bobby
 */
public class LoggingInterceptor implements InvocationHandler {
    private Object target;

    public LoggingInterceptor(Object target) {
        super();
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        // 前面日志
        System.out.println("Method " + methodName + " begins with " + Arrays.asList(args));
        
        Object result = null;
        
        try {
            result = method.invoke(target, args);
        } catch(Exception e){
            e.printStackTrace();
        }

        // 后面日志
        System.out.println("Method " + methodName + " ends with: " + result);
        return result;
    }

}
```

### 一个 Spring AOP 切面类的实现

以下是一个 Spring AOP 日志切面类。

```java
@Aspect
@Component
@Order(-1)
public class LoggingAspect {

    @Pointcut ("execution (* com.bravo.service.*.*(..))")
    public void myPointcut() {}

    // 前置通知，在目标方法开始执行前执行
    @Before("myPointcut()")
    public void beforeMethod(JointPoint jp){
        String methodName = jp.getSignature.getName();
        Object[] args = jp.getArgs();
        // 前面日志
        System.out.println("Method " + methodName + " begins with " + Arrays.asList(args));
    }

    // 后置通知，在目标方法执行后（无论目标方法是否发生异常），执行的通知。
    // 但是要注意，在后置通知中不能访问目标方法的返回结果
    @After("myPointcut()")
    public void afterMethod(JointPoint jp){
        String methodName = jp.getSignature.getName();
        // 后面日志
        System.out.println("Method " + methodName + " ends.");
    }

    // 返回通知，在方法正常结束后执行
    // 返回通知是可以访问到方法的返回值的
    @AfterReturning(pointcut = "myPointcut()", returning= "result")
    public void afterReturning (JointPoint jp， Object result) {
        String methodName = jp.getSignature.getName();
        // 后面日志
        System.out.println("Method " + methodName + " ends with: " + result);
    }

    // 异常通知，在方法抛出异常时执行。
    // 异常通知可以访问到方法抛出的异常，且可以指定在出现特定异常时再执行
    @AfterThrowing(pointcut = "myPointcut()", throwing= "e")
    public void afterThrowException (JointPoint jp， Throwable e) {
    // public afterThrowException (JointPoint jp， NullPointerException e) { // 则该方法只在抛出 NPE 的时候执行
        String methodName = jp.getSignature.getName();
        // 异常日志
        System.out.println("Method " + methodName + " throwing exception: " + e);
    }

    // 环绕通知，相当于整个动态代理方法中的全部流程
    // 注意方法参数类型不同于其他的，ProceedingJointPoint 类型的参数可以决定是否执行目标方法
    // 且环绕通知必须有返回值，返回值即为目标方法的返回值
    @Around ("myPointcut()")
    public Object aroundMethod(ProceedingJointPoint pjp) {
        Object result = null;
        String methodName = pjp.getSignature.getName();
        Object[] args = pjp.getArgs();

        try{
            // 前置通知
            System.out.println("Method " + methodName + " begins with " + Arrays.asList(args));

            // 执行目标方法
            result = pjp.proceed();

            // 返回通知
            System.out.println("Method " + methodName + " ends with: " + result);
        } catch(Exception e){
            // 异常通知
            System.out.println("Method " + methodName + " throwing exception: " + e);
        } finally {
            // 后置通知
            System.out.println("Method " + methodName + " ends.");
        }

        return result;
    } 
}
```

有一点需要注意，如果仅使用 SpringFramework (而没有使用 SpringBoot)的话，别忘了添加 `<aop:aspect-autoproxy />` 配置，自动为 Aspect 类生成代理对象。

### InvocationHandler 实现类与 AOP 切面类对照图

![AOP and Proxy](/assets/images/2020/12/AOP&Proxy.jpg)

![Throwing Advice](/assets/images/2020/12/ThrowingAdvice.jpg)

### 参考资料：
[尚硅谷首套_Spring4 视频教程](https://www.bilibili.com/video/BV1KW411u7An?p=19)