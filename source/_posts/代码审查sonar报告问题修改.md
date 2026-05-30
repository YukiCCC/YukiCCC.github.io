---
title: 代码审查sonar报告问题修改
date: 2025-03-04 14:17:03
tags:  代码审查
---

**项目引入了sonar做代码质量管理，以下为代码审查修改中反馈的相关问题：**

`` Make the enclosing method "static" or remove this set.``
**内容：**从非静态方法正确更新静态字段很难做到正确，如果有多个类实例和/或多个线程在运行，很容易导致错误。 理想情况下，静态字段仅从同步静态方法中更新。 
**解决方法：**将静态字段改为非静态。

``Remove this annotation and use constructor injection instead.``
**内容：**移除@Autowired注解使用构造器注入方式替代。
**解决方法：**
IntelliJ IDEA也会提示Field injection is not recommended，翻译成中文即：不推荐使用字段注入
可以是使用@Resource注释替换，具体区别：
**1.提供方不同**
@Autowired 是Spring提供的，@Resource 是J2EE提供的。
**2.装配时默认类型不同**
@Autowired只按type装配,@Resource默认是按name装配。
**3、使用区别**
（1）@Autowired与@Resource都可以用来装配bean，都可以写在字段或setter方法上
（2）@Autowired默认按类型装配，默认情况下必须要求依赖对象存在，如果要允许null值，可以设置它的required属性为false。如果想使用名称装配可以结合@Qualifier注解进行使用。
（3）@Resource，默认按照名称进行装配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名进行名称查找。如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。
@Resource有两个重要的属性：name和type

```Replace this call to "replaceAll()" by a call to the "replace" method ```
这句话的意思是将“replaceAll()”方法的调用替换为"replace"方法的调用。这是因为replaceAll方法是Java字符串类中的一个高级方法，它可以用正则表达式来替换字符串中的所有匹配项。然而，如果你只是想简单地替换字符串中的一个子串，那么使用 "replace" 方法会更加简洁和高效。

```Invoke method(s) only conditionally```
在编写代码时，有时需要根据特定条件来调用方法，以避免不必要的性能开销或错误。这种做法在代码质量工具如SonarLint中被称为“有条件地调用方法”（Invoke method(s) only conditionally）。这种方法可以提高代码的效率和可读性。

```Use the built-in formatting to construct this argument```
将连接字符串传递给日志记录方法也可能导致不必要的性能损失，因为每次调用该方法时都会执行连接，无论日志级别是否低到足以显示消息。应该使用内置的字符串格式而不是字符串连接，如果消息是方法调用的结果，则应该完全跳过前置条件，而应该有条件地抛出相关异常。
**反例**
```
String message = "Hellow World!";
logger.info("Test Info message："+message);
```
**正确**
```
String message = "Hellow World!";
String format = String.format("Test Info message：%s", message);
logger.info(format);

```

```Reorder the modifiers to comply with the Java Language Specification```

调整修饰符次序，反例：private final static String DD = "测试";

改成private static final String DD = "测试";

以下是修饰符的顺序：
1. Annotations 
2. public 
3. protected 
4. private 
5. abstract 
6. static 
7. final 
8. transient 
9. volatile 
10. synchronized 
11. native 
12. strictfp