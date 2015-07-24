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

This is where wildcards come in. Wildcards are a more flexible type parameter that allow for substitution (covariance and contravariance) with other types. Let's re-write that utility method with a wildcard instead:

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

In the example above, the `?` wildcard character is actually a shorthand for `? extends Object`. Object is the 'upper bound' of the type.  You can declare your own wildcard type parameters too. For example, the wildcard `Collection<? extends Fruit>` declares __Fruit__ as the __upper__ bound of the type. This permits covariant assignments:

```java
Collection<? extends Fruit> fruit1 = new ArrayList<Apple>();//compiles
Collection<? extends Fruit> fruit2 = new ArrayList<Orange>();//compiles
Collection<? extends Fruit> fruit3 = new ArrayList<Orange>();//compiles

Collection<? extends Fruit> fruit4 = new ArrayList<Plant>();//won't compile!!
```

### Wildcards with __super__

The second wildcard types uses the `super` keyword which establishes a __lower__ bound of the type. This type is contravariant with any collection of Fruit types or supertypes:

```java
Collection<? super Fruit> fruit1 = new ArrayList<Fruit>();//compiles
Collection<? super Fruit> fruit2 = new ArrayList<Plant>();//compiles
Collection<? super Fruit> fruit3 = new ArrayList<Object>();//compiles

Collection<? super Fruit> fruit4 = new ArrayList<Orange>();//won't compile!!
```

### The Get and Put Principle

The wildcard types can be used anywhere you use a generic type (e.g. method or class declarations). They have a very profound impact on the operations you can perform with a collection. 

Recall that Arrays in Java are covariant, and this results in the potential for Runtime errors that cannot be caught at compile time:

```java
Apple[] apples = new Apple[]{new Apple(), new Apple()};
makeSmoothie(apples);

public void makeSmoothie(Fruit[] fruit) {
    Fruit aFruit = fruit[0];//type-safe
    ....
    fruit[0] = new Orange();//runtime-time error  
    blend(fruit);
}
```

It is safe to get a Fruit reference from the fruit array, but it isn't safe to put an instance of a Fruit into the array (because it is actually an array of Apples). The people designing the Java language (and Generics) did not permit this same mistake with typed collections. For a collection reference defined with an `extends` wildcard type, the compiler prevents you from adding an element:

```java
List<Apple> apples = new ArrayList<Apple>();
apples.add(new Apple());
makeSmoothie(apples);

public void makeSmoothie(List<? extends Fruit> fruit) {
    Fruit aFruit = fruit.get(0);//type-safe
    fruit.add(new Orange());//compile-time error
}
```

The compiler doesn't know if the `fruit` parameter references a Collection of Apples, Oranges, or Fruit (or a maybe a combination of all three?). As a result:

- It's safe to read elements from the collection __as__ type Fruit (cause an Apple is a Fruit, an Orange is a Fruit, etc)
- It isn't safe to write __anything__ to the collection because the compiler cannot know the underlying collection type at runtime. The collection object could be an ArrayList<Apple> or a LinkedList<Orange>, etc. As you can see, we avoid the problem that exists with arrays.

Likewise, the compiler will not allow you to get an element from a Collection defined with the wildcard `? super Type`.  Remember the problem with Collections before Generics:

```java
List list = new ArrayList();
list.add("one");
list.add(1);
String s = (String) list.get(1);//ClassCastException
```

The Java compiler prevents this problem with collection references declared with the `super` wildcard type. The compiler will allow you __put__ elements in, but not __get__ elements out.

```java
List<Plant> plants = new  ArrayList<Plant>();
plants.add(new Plant());//a Plant is a Plant
plants.add(new Fruit());//a Fruit is a Plant
plants.add(new Apple());//an Apple is a plant

List<? super Fruit> fruit = plants;//contravariant types
fruit.add(new Fruit());//still ok 
fruit.add(new Apple());//still ok

//Not permitted by the compiler, and good thing 
//because it is not a Fruit! It's a Plant!!
Fruit fruityGoodness = fruit.get(0);//won't compile
```

OK, you are allowed get one type out of a Collection using the super wildcard:
  
```java
Object obj = fruit.get(0);
```

The compiler will allow this because it can be sure that __every__ reference type is an Object. Just don't forget, the reference could also be null (but that's nothing new).

All of the above can be summed up via the __Get and Put Principle__:

>Use an extends wildcard when you only get values out of a structure, user a super wildcard when you only put values into a structure, 
>and don't use a wildcard when you both get and put. (Naftalin & Wadler, [Java Generics and Collections](http://oreilly.com/catalog/9780596527754), pg 19).

Even if nothing else in this post makes sense to you, remember the __Get and Put Principle__! It is __inviolable__.... even if it doesn't seem... you know... intuitive.
