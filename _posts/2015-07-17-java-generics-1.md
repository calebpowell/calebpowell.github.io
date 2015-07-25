---
layout: post
title: Making Sense of Java Generics Pt. 1
---

Java Generics have been around for awhile now. Anyone programming in Java on a regular basis should be comfortable with how they work. Except, sometimes they can be a little tricky. Can't they? This post attempts to give you a better mental model of generics in Java by explaining the basics around Generics and _why_ they sometimes seem a little weird (or simply don't want to work for you). Writing this stuff out also helps me to internalize it ;-). 

Most of the information in this post is based on the following resources (check them out for a deeper understanding of the material discussed here):

* [Maurice Naftalin & Philip Wadler, Java Generics and Collections (California: O'Reilly Media, 2007)](http://oreilly.com/catalog/9780596527754 "Java Generics and Collections")
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

If you have been programming in Java, most of the examples above should be familiar to you.

### Arrays and Collections (before Generics)

Before looking at Generics, let's consider complex types like arrays and Collections with respect to variance. Java arrays are covariant:

```java
//assignment
Object[] objs = new String[]{"a", "b"};

//method invocation
public static void doSomething(Object[] objs) {};
doSomething(new String[]{"a", "b"});
```
The fact that arrays are covariant means that the compiler cannot guarantee type safety when you _put_ an element into an array. For example, the following code will compile but result in an ArrayStoreException at runtime:

```java
Object[] objs = new String[]{"a", "b"};

Object a = objs[0];//it is safe to 'get' an object from the array

objs[1] = new Integer(1);//not so safe to put. this compiles, but results in the ArrayStoreException
```

Yeah, not so good when your language is trying to guarantee type safety.

What about Collections? Unlike arrays, pre-generic Java Collections did not allow you to specify the component type (that is where generics come in). As a result, you had to perform a cast whenever you retrieved an item from a collection:

```java
List strings = new ArrayList();
strings.add('hello');

String s = (String) strings.get(0);//cast is required for the compiler
```

Of course, things could go wrong and someone could pass you a collection whose component types don't match your expectations:

```java
List ostensiblyStrings = new ArrayList();
ostensiblyStrings.add(new Integer(1));

String s = (String) ostensiblyStrings.get(0);//ClassCastException at runtime!
```

So, this was a problem (and generics address it). Also, keep in mind that Collections themselves are just reference types and therefore covariant:

```java
//covariant assignment
Collection list = new ArrayList();

//covariant method invocation
public static void doSomething(Collection objs) {};
doSomething(list);
```

### Enter Generics

Generics allow us to abstract over reference types and enhance type-safety. They are implemented using a technique called erasure, whereby the compiler erases the type parameters (e.g. <String>) and inserts the casts that programmers used to add manually. It is important to understand this; the generic types are not compiled into the bytecode but are erased by the compiler (after it uses them to ensure that your code is type safe). The casts created by the compiler are guaranteed not to fail at runtime provided there are no raw types. Your compiler or IDE will provide warnings when it encounters raw types.  Fix them.

Before going any further, let's consider a simple class hierarchy that we can use to illustrate our types:

```java
public class Animal{};
public class Bird extends Animal{};
public class Duck extends Bird{};
public class Goose extends Bird{};
```

As mentioned above, reference types are covariant so you can substitute a __subtype__ where a __supertype__ is expected:

```java
Bird bird = new Duck();
```

Raw Collections (Collections whose component type isn't specified) are also covariant:

```java
Collection col = new ArrayList();
```

However, Collections specified with different type parameters are invariant! So the following code will not compile:

```java
Collection<Bird> col = new ArrayList<Duck>();//won't compile!
```

This code can only be made to compile by removing the invariance on the type paramaters on either the left or right side:

```java
Collection<Duck> col = new ArrayList<Duck>();//compiles!
```

In other words:

A Duck is a Duck:

```java
Duck duck = new Duck();
```

A Duck is a Bird:

```java
Bird bird = new Duck();
```

A List of Duck is a Collection of Duck:

```java
Collection<Duck> col = new ArrayList<Duck>();
```

But, a List of Duck is not a List of Bird! 

```java
ArrayList<Bird> col = new ArrayList<Duck>();//won't compile
```

The invariance of generic types can often feel counter-intuitive. The fact that Generic types are invariant means that some types of assignment are impossible. Consider the following utility method to print the components of a collection:

```java
public static void printElements(Collection<Object> col) {
    for (Object obj : col) {
        System.out.println(obj);
    }
}

Collection<Duck> ducks = new ArrayList<Duck>();
printElements(ducks);//won't compile!!
```

In the [next post](../java-generics-2/), we will take a look at how wild cards can help provide some covariant and contravariant assignments when using generic types.
