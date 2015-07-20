---
layout: post
title: Making Sense of Java Generics Pt. 2
---

This is part 2 in my post on Making sense of Java Generics. Anyone programming in Java on a regular basis should be comfortable with how they work. Most of the information in this post is based on the following resources (check them out for a deeper understanding of the material discussed here):

* [Maurice Naftalin & Philip Wadler, Java Generics and Collections (California: O'Reilly Media, 2007)](http://oreilly.com/catalog/9780596527754 "Java Generics and Collections")
* [Gilad Bracha, Lesson: Generics](http://docs.oracle.com/javase/tutorial/extra/generics/index.html "Lesson: Generics")
* [Covariance and Contravariance (Wikipedia)](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science) "Covariance and Contravariance")

We finished up part #1 with a discussion around the invariance of generic types, and gave an example of a utility method that cannot be invoked with any of our sample types (except for Objects):

```java
public static void printElements(Collection<Object> col) {
    for (Object obj : col) {
        System.out.println(obj);
    }
}

Collection<Apple> apples = new ArrayList<Apple>();
printElements(apples);//won't compile!!
Collection<Object> objs = new ArrayList<Object>();
printElements(objs);//this will compile
```

### Wildcards

This is where wildcards come in. Wildcards are a more flexible type parameter that allow for substitution (covariancs and contravariance) with other types. Let's try that utility method with a wildcard instead:

```java
public static void printElements(Collection<?> col) {
    for (Object obj : col) {
        System.out.println(obj);
    }
}

Collection<Apple> apples = new ArrayList<Apple>();
printElements(apples);//compiles!!
Collection<Orange> oranges = new ArrayList<Orange>();
printElements(oranges);//compiles!!
```

By changing the col parameter from the type `Collection<Object>` to the type `Collection<?>`, we have introduced covariance and made it possible to invoke this utility method with any generic type parameter. 

### Wildcards with __extends__

In the example above, the `?` wildcard character is actually a shorthand for `? extends Object` which roughly translates to __"I accept any type parameter that extends the Object type"__. Object is the upper bound of the type. 

You can declare your own wildcard type parameters too. For example, the wildcard `Collection<? extends Fruit>` declares __Fruit__ as the upper bound of the type. This permits covariant assignments:

```java
Collection<? extends Fruit> fruit1 = new ArrayList<Apple>();
Collection<? extends Fruit> fruit2 = new ArrayList<Orange>();
```

### Wildcards with __super__
