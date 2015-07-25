---
layout: post
title: Making Sense of Java Generics Pt. 2
---

This is part 2 in my post on Making sense of Java Generics. Anyone programming in Java on a regular basis should be comfortable with how they work. Most of the information in this post is based on the following resources (check them out for a deeper understanding of the material discussed here):

* [Maurice Naftalin & Philip Wadler, Java Generics and Collections (California: O'Reilly Media, 2007)](http://oreilly.com/catalog/9780596527754 "Java Generics and Collections")
* [Gilad Bracha, Lesson: Generics](http://docs.oracle.com/javase/tutorial/extra/generics/index.html "Lesson: Generics")
* [Covariance and Contravariance (Wikipedia)](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science) "Covariance and Contravariance")

We finished up [Part #1](../java-generics-1/) with a discussion around the invariance of generic types, and gave an example of a utility method that cannot be invoked with any of our sample types (except for Objects):

```java
public static void printElements(Collection<Object> col) {
    for (Object obj : col) {
        System.out.println(obj);
    }
}

Collection<Duck> ducks = new ArrayList<Duck>();
printElements(ducks);//won't compile!!
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

Collection<Duck> ducks = new ArrayList<Duck>();
printElements(ducks);//compiles!!
Collection<Goose> geese = new ArrayList<Goose>();
printElements(geese);//compiles!!
```

By changing the col parameter from the type `Collection<Object>` to the type `Collection<?>`, we have introduced covariance and made it possible to invoke this utility method with any generic type parameter. 

### Wildcards with _extends_

In the example above, the `?` wildcard character is actually a shorthand for `? extends Object`. Object is the _upper bound_ of the type.  You can declare your own wildcard type parameters too. For example, the wildcard `Collection<? extends Bird>` declares _Bird_ as the upper bound. This permits covariant assignments:

```java
Collection<? extends Bird> bird1 = new ArrayList<Duck>();//compiles
Collection<? extends Bird> bird2 = new ArrayList<Goose>();//compiles
Collection<? extends Bird> bird3 = new ArrayList<Bird>();//compiles

Collection<? extends Bird> bird4 = new ArrayList<Animal>();//won't compile!!
```

### Wildcards with _super_

The second wildcard types uses the `super` keyword which establishes a _lower bound_ of the type. This type is contravariant with any collection of Bird types or supertypes:

```java
Collection<? super Bird> bird1 = new ArrayList<Bird>();//compiles
Collection<? super Bird> bird2 = new ArrayList<Animal>();//compiles
Collection<? super Bird> bird3 = new ArrayList<Object>();//compiles

Collection<? super Bird> bird4 = new ArrayList<Goose>();//won't compile!!
```

### The Get and Put Principle

The wildcard types can be used anywhere you use a generic type (e.g. method or class declarations). They have a very profound impact on the operations you can perform with a collection. 

Recall that Arrays in Java are covariant, and this results in the potential for Runtime errors that cannot be caught at compile time:

```java
Duck[] ducks = new Duck[]{new Duck(), new Duck()};
makeTurducken(ducks);

public void makeTurducken(Bird[] birds) {
    Bird aBird = birds[0];//type-safe
    ....
    birds[0] = new Goose();//runtime-time error  
    blend(birds);
}
```

It is safe to get a Bird reference from the bird array, but it isn't safe to put an instance of a Bird into the array in this example (because the method is actually being passed an array of Duck). The people designing the Java language (and Generics) did not permit this same mistake with typed collections. For a collection reference defined with an `extends` wildcard type, the compiler prevents you from adding an element:

```java
List<Duck> ducks = new ArrayList<Duck>();
ducks.add(new Duck());
makeTurducken(ducks);

public void makeTurducken(List<? extends Bird> bird) {
    Bird aBird = bird.get(0);//type-safe
    ....
    bird.add(new Goose());//compile-time error
    blend(birds);
}
```

The compiler doesn't know if the `bird` parameter references a Collection of Duck, Goose, or Bird (or a maybe a combination of all three?). As a result:

- It's safe to read elements from the collection _as_ type Bird (cause a Duck is a Bird, a Goose is a Bird, etc)
- It isn't safe to write _anything_ to the collection because the compiler cannot know the underlying collection type at runtime. The collection object could be an ArrayList<Duck> or a LinkedList<Goose>, etc. As you can see, we avoid the problem that exists with arrays.

Likewise, the compiler will not allow you to get an element from a Collection defined with the wildcard `? super Type`.  Remember the problem with Collections before Generics:

```java
List list = new ArrayList();
list.add("one");
list.add(1);
String s = (String) list.get(1);//ClassCastException
```

The Java compiler prevents this problem with collection references declared with the `super` wildcard type. The compiler will allow you _put_ elements in, but not _get_ elements out.

```java
List<Animal> animals = new  ArrayList<Animal>();
animals.add(new Animal());//an Animal is an Animal
animals.add(new Bird());//a Bird is an Animal
animals.add(new Duck());//a Duck is an animal

List<? super Bird> birds = animals;//contravariant types
birds.add(new Bird());//still ok because a Bird is an animal
birds.add(new Duck());//still ok because a Duck is an animal

//Not permitted by the compiler, and good thing 
//because it is not a Bird! It's an Animal!!
Bird birdy = birds.get(0);//won't compile
```

OK, you are allowed get one type out of a Collection using the super wildcard:

```java
Object obj = bird.get(0);
```

The compiler will allow this because it can be sure that _every_ reference type is an Object. Just don't forget, the reference could also be null (but that's nothing new).

All of the above can be summed up via the _Get and Put Principle_:

>Use an extends wildcard when you only get values out of a structure, user a super wildcard when you only put values into a structure, 
>and don't use a wildcard when you both get and put.    [_(Naftalin & Wadler, pg 19)_][1] 

Even if nothing else in this post makes sense to you, remember the _Get and Put Principle_! It is _inviolable_.... even if it doesn't seem... you know... intuitive.

### Wildcard Usage

Acceptable usage of the wildcard type includes:

- Variable declaration

```java
List<? extends Animal> animals = new ArrayList<>();
```

- Method Parameters

```java
public void doSomething(List<? super Bird> bird) {

}
```

- _Nested_ new instance declarations

```java
new ArrayList<List<? extends Bird>>();
```

- _Nested_ class declarations

```java
public class MyList extends ArrayList<List<?>> {
}
```

- _Nested_ generic method invocations

```java
public  class ListFactory {
    public static <T> List<T> createList() {
        return new ArrayList<T>();
    }
}

List<?> list = ListFactory.<?>createList();//won't compile
List<?> list = ListFactory.<List<?>>createList();//compiles
```

### Wilcard Restrictions

- Non-nested instance creation

```java
new ArrayList<? extends Bird>();//won't compile
```

- Non-nested Class Declarations

```java
public class MyList extends ArrayList<?> {
}//won't compile
```

- Non-nested Generic method invocations

```java
List<?> list = ListFactory.<?>createList();//won't compile
```

[1]: http://oreilly.com/catalog/9780596527754
