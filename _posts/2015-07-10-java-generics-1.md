---
layout: post
title: Making Sense of Java Generics Pt. 1
---

Java Generics have been around for awhile now. Anyone programming in Java on a regular basis should be comfortable with how they work. Except, sometimes they can be a little tricky. Can't they? This post attempts to give you a better mental model of generics in Java by explaining the basics around Generics and _why_ they sometimes seem a little weird (or simply don't want to work for you). Most of the information in this post is based on the following resources (you should check them out for a deeper understanding of the material discussed here):

* [Maurice Naftalin & Philip Wadler, Java Generics and Collections (California: O'Reilly Media, 2007) Java Generics and Collections](http://oreilly.com/catalog/9780596527754 "Java Generics and Collections")
* [Gilad Bracha, Lesson: Generics](http://docs.oracle.com/javase/tutorial/extra/generics/index.html "Lesson: Generics")
* [Covariance and Contravariance (Wikipedia)](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science) "Covariance and Contravariance")

### Covariance and Contravariance

Let's refresh our understanding around the concepts of covariance, contravariance, and how they relate specifically to Arrays and Collections in Java.

__Covariance:__ Converting from wider to narrower. Covariance permits the use of a subtype where a supertype is expected. The following are all examples of covariance:

```java
//covariant assignment
Object obj = new String("");
int n = (int) 2L;

//Covariant method invocation
public static void doSomething(Object obj) {};
doSomething(new String(""));
doSomething(new Integer(123));

//Covariant return types (added in Java 1.5)
class Parent {
    public Object doSomething(Object obj) {...};
}

class Child extends Parent {
    public String doSomething(Object obj) {...};
}
```

__Contravariance:__ Converting from narrower to wider. Contravariance permits the use of a wider type where a narrower type is expected. The following are all examples of contravariance:

```java

//contravariant assignment
int i = 0;
long n = i;

//overriding a method with a narrower argument
class Parent {
    public void doSomething(String obj) {...};
}

class Child extends Parent {
    public void doSomething(Object obj) {...};
}
```

__Invariance:__ Not able to convert. For example, the following code will not compile:

```java
boolean foo = 1;//won't compile
String s = new Integer(1);//won't compile
```

If you have been programming in Java, most of the above should be familiar to you.

