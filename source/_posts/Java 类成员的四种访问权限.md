---
title: Java访问权限修饰符
date: 2021-09-21
tags: [java]
---



# 访问权限修饰符

Java 为 类、属性字段、方法 提供了四种访问权限 : default, public, private, protected

## default (friendly) - 包访问权限

> 是默认访问权限, 没有任何关键字

Java 中不声明任何访问权限修饰符则默认为此, 包内其他成员可访问

- 其他包的子类不可见
- 若两个类存放在同一个目录下且没有设定包名称, Java 自动将它们看作隶属于该目录的默认包中

## public - 接口访问权限

对任何人都可访问

## private - 无法访问

任何其他类无法访问

- 子类不可见

## protected - 继承访问权限

相同包可见, 子类可见

