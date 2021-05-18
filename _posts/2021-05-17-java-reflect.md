---
title: Java反射笔记
tag:
  - java
typora-root-url: ../

---

记一下比较常用到的，又容易忘的

<!--more-->

[TOC]

## getInterfaces & getGenericInterfaces

获取类实现的接口列表，后者会进一步携带接口拥有的参数化类型信息

可以用这个方法获得类继承接口时，在接口泛型参数处填入的实际类型

## getTypeParameters

获得一个类声明的泛型参数列表，如`class A<T, K> {}`会返回`[T, K]`，类型为`TypeVariable`

## 获得对象的泛型信息

类似于

```java
public class Test<T> {
    public void fun(T t){}
}
```

似乎无法得到一个实例他的泛型参数具体类型，此时通过`method.getParameterTypes()`得到的是`Object`

如果是这种

```java
public class Test<T> {
    public void fun(List<T> t){}
}
```

得到的是`List`，用`method.getGenericParameterTypes()`也只能得到`T`

如果是这种，可以通过`method.getGenericParameterTypes()`得到`String`的信息

```java
public void fun(List<String> t){}
```

## 泛型接口在不确定实现使用的类型参数的情况下，怎么调用含泛型的方法？

```java
interface Functional<T>{
    void call(T t);
}

class Function implements Functional<A> {
    public void call(A t){...}
}

Functional<?> functional = new Function(); // 具体使用中不知道是哪一种实现
A a = new A();
// functional.call(a); 报错
```

涉及？的只能用来交接不能使用？

[泛型陷阱](https://www.cnblogs.com/hongdada/p/10683795.html)