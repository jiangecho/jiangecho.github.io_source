title: Note of OReilly Java in a Nutshell 6th Edition
date: 2014-01-06 23:01:42
tags:
---

# Java Syntax from the Ground Up

###Labeled statement
A labeled statement is simply a statement that has been given a name by prepending an identifier and a colon to it. Labels are used by the **break** and **continue** statements. For example:

```
rowLoop: for(int r = 0; r < rows.length; r++) {        // Labeled loop
   colLoop: for(int c = 0; c < columns.length; c++) {  // Another one
     break rowLoop;                                    // Use a label
   }
}

```

###The try-with-resources Statement
Java (since version 7) provides a very useful mechanism for automatically closing resources that require cleanup. This is known as try-with-resources, or **TWR**
```
try (InputStream is = new FileInputStream("/Users/ben/details.txt")) {
  // ... process the file and do not need to write catch and finally
}
```
###Checked and Unchecked Exceptions
**Checked exceptions** arise in specific, well-defined circumstances, and very often are conditions from which the application may **be able to partially or fully recover**.
Any exception object that is an `Error` is unchecked. There is also a subclass of `Exception` called `RuntimeException`！and any subclass of `RuntimeException` is also an unchecked exception. All other exceptions are checked exceptions.

###Variable-Length Argument Lists
```
public static int max(int first, int... rest/*can only be the last parameter*/) {
    /* body omitted for now */
}

...
max(1);
max(1, 2, 3);
max(1, 2, 3, 4);
...
```
yeah, this method is the same as:
```
public static int max(int first, int[] rest) {
    /* body omitted for now */
}

...
max(1, new int[]{2, 3, 4});
...
```

###Lambda Expressions
A **lambda expression** is essentially a *function* that does not have a name, and can be treated as a value in the language.

in Java, this means that a lambda is an **anonymous method** that is defined on some class (that is possibly unknown to the developer).

The syntax for a lambda expression looks like this:
```
( paramlist ) -> { statements }
```
eg:
```
Runnable r = () -> System.out.println("Hello World");
```

**Attention:**

When a lambda expression is used as a value it is automatically converted to a new object of the correct type for the variable that it is being placed into.

###Initializer blocks

* static initializer
* instance initializer

###Interface
An interface is intended to represent only an API！so it provides a description of a type, and the methods (and signatures) that classes that implement that API should provide.

In general, a Java interface does not provide any implementation code for the methods that it describes. These methods are considered mandatory！any class that wishes to implement the interface must provide an implementation of these methods.

However, an interface may wish to mark that some API methods are optional, and that implementing classes do not need to implement them if they choose not to.  This is done with the `default` keyword！and the interface must provide a default implementation of these optional methods, which will be used by any implementation that elects not to implement them.

###Generics
To indicate that a type is a container that holds instances of another reference type we enclose the payload type that the container holds within angle brackets:
```
List<CenteredCircle> shapes = new ArrayList<CenteredCircle>();
```
Container types are usually called `generic types`！and they are declared like this:
```
interface Box<T> {
  void box(T t);
  T unbox();
}
```
This indicates that the `Box` interface is a general construct, which can hold any type of payload. It isn¨t really a complete interface by itself！it¨s more like a general description of a whole family of interfaces, one for each type that can be used in place of `T`.

The syntax <T> has a special name！it¨s called a `type parameter`, and another name for a generic type is a `parameterized type`. 

A parameterized type, such as `ArrayList<T>`, is not instantiable！we cannot create instances of them. This is because `<T>` is just a type parameter！merely a placeholder for a genuine type. It is only when we provide a concrete value for the type parameter, (e.g., `ArrayList<String>`), that the type becomes fully formed and we can create objects of that type.

This poses a problem if the type that we want to work with is unknown at compile time. Fortunately, the Java type system is able to accommodate this concept. It does so by having an explicit concept of the unknown type！which is represented as `<?>`.

This is the simplest example  of Java¨s wildcard types.
We can write expressions that involve the unknown type:
```
ArrayList<?> mysteryList = unknownList();
Object o = mysteryList.get(0);
```
This is perfectly valid Java！`ArrayList<?>` is a complete type that a variable can have, unlike `ArrayList<T>`.

###Enums and Annotations

###Singleton
```
public class Singleton {
    private final static Singleton instance = new Singleton();
    private static boolean initialized = false;
    // Constructor
    private Singleton() {
      super();
    }
    private void init() {
      /* Do initialization */
    }
    // This method should be the only way to get a reference 
    // to the instance
    public static synchronized Singleton getInstance() {
      if (initialized) return instance;
      instance.init();
      initialized = true;
      return instance;
    }
}
```





