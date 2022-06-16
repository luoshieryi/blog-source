---
title: Java 异常处理
date: 2021-10-18
tags: [java, error]
---

# 异常处理

## try-catch-finally 的调用顺序

try 中未抛出异常则不执行 catch 中的方法
即使 try 或 catch 中有 return 语句仍会执行 finally 中的语句, 且 finally 中的 return 会覆盖 try\catch 的return

## test方法

```java
package com.company;

public class tryCatchFinally {

    public String test(int a) {
        try {
            System.out.print("try\t\t");
            throw new Exception();
        } catch (Exception e) {
            System.out.print("catch\t\t");
            return "catch";
        } finally {
            System.out.print("finally\t\t");
            if (a == 1) {
                return "finally";
            }
        }
    }

}
```

main方法 (逃) 

```java
public class Main {

    public static void main(String[] args) throws Exception{

        System.out.println( Class.forName("com.company.tryCatchFinally").getDeclaredMethod("test", int.class).invoke(Class.forName("com.company.tryCatchFinally").newInstance(), 1) );

        System.out.println( Class.forName("com.company.tryCatchFinally").getDeclaredMethod("test", int.class).invoke(Class.forName("com.company.tryCatchFinally").newInstance(), 0) );

    }
}
```

输出结果

```java
try		catch		finally		finally
try		catch		finally		catch
```

