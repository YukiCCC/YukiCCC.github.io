---
title: 基于AOP的日志系统
date: 2024-09-07 01:08:25
tags:
---
## 基于AOP的日志模块
本文旨在分析参与项目中的日志，如何基于AOP面向切面原理实现controller请求及响应信息打印。本项目将日志记录的逻辑封装成一个切面，然后通过切入点和通知来指定在哪些方法需要执行日志记录的操作。
### AOP中注解的含义
@Aspect：切面。表示一个横切进业务的一个对象。它里面包含切入点(Pointcut)和Advice（通知）。
@Pointcut：切入点。表示需要切入的位置，比如某些类或者某些方法，也就是先定一个范围。
@Before：Advice（通知）的一种，切入点的方法体执行之前执行。
@Around：Advice（通知）的一种，环绕切入点执行也就是把切入点包裹起来执行。
@After：Advice（通知）的一种，在切入点正常运行结束后执行。
@AfterReturning：Advice（通知）的一种，在切入点正常运行结束后执行，异常则不执行
@AfterThrowing：Advice（通知）的一种，在切入点运行异常时执行。

以下是一个类似案例：

### 日志注释
```java
// 日志注解
@Target({ElementType.PARAMETER,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Log {

    /**
     * 描述
     */
    String description() default "";

    /**
     * 方法类型 INSERT DELETE UPDATE OTHER
     */
    MethodType methodType() default MethodType.OTHER;
}
```
其中@Target用于指定自定义注解可以应用到哪些Java元素上。它的值是一个或多个 ElementType 枚举常量。你提到的 {ElementType.PARAMETER, ElementType.METHOD} 表示该注解可以应用到方法参数（PARAMETER）和方法（METHOD）上。

ElementType.PARAMETER: 该注解可以应用于方法参数上。
ElementType.METHOD: 该注解可以应用于方法上。

@Retention 指定了注解的生命周期，也就是注解信息在什么阶段可以被保留。它的值是一个 RetentionPolicy 枚举常量。

RetentionPolicy.RUNTIME: 表示该注解不仅会被编译到字节码中，还会在运行时保留，因此可以通过反射机制获取到该注解信息。这在AOP中非常常见，因为AOP需要在运行时根据注解来执行相应的逻辑。

### 日志切面
```java
// 日志切面
@Component
@Aspect
public class LogAspect {
  // 切入点，所有被 Log 注解标注的方法
  @Pointcut("@annotation(cn.javaguide.annotation.Log)")
  public void webLog() {
  }
```
其中@Aspect 是 Spring AOP（Aspect-Oriented Programming，面向切面编程）中的一个注解，用于定义一个类为“切面”（Aspect）。在 Spring AOP 中，切面是用来实现横切关注点的模块化，它将横切关注点（如日志记录、事务管理、安全性检查等）从业务逻辑中分离出来，以提高代码的可维护性和可重用性。

@Aspect 注解的作用
定义切面：@Aspect 标识一个类为切面类，这个类包含了与横切关注点相关的代码逻辑，如方法拦截、日志记录等。
包含切入点和通知：一个切面类通常包含一个或多个切入点（Pointcut）和通知（Advice）。切入点定义了在哪些地方应用横切关注点，而通知则定义了在切入点执行的具体动作。

### 环绕通知 
```java
   /**
   * 环绕通知
   */
  @Around("webLog()")
  public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
    // 省略具体的处理逻辑
  }

  // 省略其他代码
}
```
其中@Around：Advice（通知）的一种，环绕切入点执行也就是把切入点包裹起来执行。

### 两套日志系统
本项目使用的是@Pointcut()注释，对项目controller目录下的controller接口进行扫描，以进行控制