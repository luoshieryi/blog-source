---
title: 写一个Java注解与 AOP
date: 2021-10-31
tags: [java, spring, springboot, reflect]
---

# 写一个 Java 注解与 AOP

## 元注解

元注解: 用在其他注解上的注解

四种元注解: 

- @Target : 描述注解的使用范围
- @Retention : 描述注解的生命周期
- @Documented : 将此注解包含在Javadoc中
- @Inherited : 使被它修饰的注解具有继承性

### @Target

描述注解的可使用范围 (取值范围在 java.lang.annotation.ElementType.java 中)

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```

ElementType :

```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,		//类，接口（包括注解类型）或枚举的声明

    /** Field declaration (includes enum constants) */
    FIELD,		//属性的声明

    /** Method declaration */
    METHOD,		//方法的声明

    /** Formal parameter declaration */
    PARAMETER,		//方法形式参数声明

    /** Constructor declaration */
    CONSTRUCTOR,		//构造方法的声明

    /** Local variable declaration */
    LOCAL_VARIABLE,		//局部变量声明

    /** Annotation type declaration */
    ANNOTATION_TYPE,		//注解类型声明

    /** Package declaration */
    PACKAGE,		//包的声明

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```

### @Retetion

限定注解使用在其他类后, 可以保留到什么时候 (取值定义在 java.lang.annotation.RetentionPolicy.java 中)

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```

RetentionPolicy : 

```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,		//源文件保留(javac把.java源文件编译成.class时，就将相应的注解去掉), 只在编译时用到, 编译后无意义

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,		//编译期保留(注解被保留到class文件，但jvm加载class文件时候被遗弃), 是默认值

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME		//运行期保留, 自定义注解一般都用这个
}
```

### @Documented

将此注解包含在Javadoc中

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```

### @Inherited

如果某个类使用了被@Inherited修饰的注解，则其子类将自动具有该注解

- 仅对 @Target(ElementType.ANNOTATION_TYPE) 的注解有效
- 仅对 class 的继承 (对 interface 的继承无效)

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

## 自定义一个注解

### 通过反射获取注解

反射 : 程序在运行期可以拿到一个对象的所有信息; 通过 Class 实例获取 class 信息
		使程序在运行期对某实例一无所知的情况下, 调用它的方法

#### 定义一个注解

```java
package com.company;

import java.lang.annotation.*;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface Painter {

    String occupation() default "";
    String twitter() default "";

}
```

#### 定义一个实体类

```java
package com.company;

public class Painters {

    @Painter(occupation = "幼术师", twitter = "@shiratamacaron")
    private String shiratama = "shiratama";

    @Painter(twitter = "@xdeyuix")
    private Integer deyui = 1220;

    private String yusano = "yusano";

    public String getShiratama() {
        return shiratama;
    }

    public void setShiratama(String shiratama) {
        this.shiratama = shiratama;
    }

    public Integer getDeyui() {
        return deyui;
    }

    public void setDeyui(Integer deyui) {
        this.deyui = deyui;
    }

    public String getYusano() {
        return yusano;
    }

    public void setYusano(String yusano) {
        this.yusano = yusano;
    }
}
```

#### 通过反射获取注解与字段值

```java
package com.company;

import java.lang.reflect.Field;

public class Main {

    public static void main(String[] args) throws Exception{

        //创建类模板
        Class<?> c = Class.forName("com.company.Painters");

        //创建类的实例
        Object object = c.newInstance();

        //获取所有字段
        for (Field f : c.getDeclaredFields()) {

            f.setAccessible(true);      //设置字段可访问, 否则无法访问private字段, f.get 时报错 IllegalAccessException
            System.out.println("当前字段" + f.getName());		//输出当前字段

            //判断这个字段是否有Painter注解
            if (f.isAnnotationPresent(Painter.class)) {
                Painter annotation = f.getAnnotation(Painter.class);
                System.out.println(" 有@Painter注解" +
                            "        职业: "  + annotation.occupation() +
                            "        推特: "  + annotation.twitter() +
                            "        值: "    + f.get(object));
            } else {
                System.out.println((" 无@Painter注解" + "          值: " + f.get(object)));
            }

        }

    }
}
```

输出结果 : 

```java
当前字段shiratama
 有@Painter注解        职业: 幼术师        推特: @shiratamacaron        值: shiratama
当前字段deyui
 有@Painter注解        职业:         推特: @xdeyuix        值: 1220
当前字段yusano
 无@Painter注解          值: yusano
```

### 注解与 Spring AOP

在日志中print一点东西.....~~再修改一下返回值)咕了~~

#### 引入 Spring AOP 依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

#### 注解类

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface GetPainterInfo {
}
```

#### 切面类 (更多见注释)

```java
@Aspect     //声明是切面类
@Component  //声明是组件类, 注册bean到Spring容器中
public class PainterAspect {

    //@Pointcut 表示这是一个切点
    //@annotation 表示此切点切到一个注解上, 之后是该注解的全类名
    @Pointcut("@annotation(com.example.demo.GetPainterInfo)")
    public void painterPointCut(){}

    //环绕通知, 表示在painterPointCut(即@Painter注解的方法)前后做一点事情
    @Around("painterPointCut()")
    public Object painterAround(ProceedingJoinPoint joinPoint){

        //获取方法的名称
        String methodName = joinPoint.getSignature().getName();
        //获取传入参数名称
        String[] paramName = ( (MethodSignature)joinPoint.getSignature() ).getParameterNames();
        //获取传入参数值
        Object[] param = joinPoint.getArgs();

        //遍历所有参数名称添加到StringBuilder对象中
        StringBuilder nameStringBuilder = new StringBuilder();
        for (String string : paramName) {
            nameStringBuilder.append(string + "; ");
        }
        StringBuilder paramStringBuilder = new StringBuilder();
        for (Object object : param) {
            paramStringBuilder.append(object + "; ");
        }
        //直接print到日志
        System.out.println ("进入["+ methodName + "]方法\n" +
                            "参数名称[" + nameStringBuilder + "]\n" +
                            "参数值[" + paramStringBuilder + "]");

        try {
            //执行原方法并获取返回值
            Object result = joinPoint.proceed();
            //@Around后原方法的返回会被接管, 此处不返回的话就没有返回值了
            return result;
        } catch (Throwable e) {
            e.printStackTrace();
        }

        //凑数用的返回 ↓↓
        return new Object();
    }

}
```

运行结果: 

![image-20210924010844542](https://cdn.jsdelivr.net/gh/luoshieryi/images@main//markdown/image-20210924010844542.png)
![image-20210924010955569](https://cdn.jsdelivr.net/gh/luoshieryi/images@main//markdown/image-20210924010955569.png)

