---
layout: post
title:  "Quick Intor About Java Records"
date:   2024-01-02 12:27:06 +0100
categories: java update
---


## Quick intro about Java Records
Java records are special classes with some pretty powerful capabilities. They're immutable and 
particularly useful for creating simple data carrier classes.

Traditionally, in Java, we acquire to write a lot of boilerplate code like constructors, `equals()`, `hashcode()`, 
and `toString()` methods to create a simple data-carrier class. This is often too verbose, repetitive
and requires verbose class definitions such as these. 

```java
public class Author {
    private final String firstName;
    private final String lastName;
    // Constructors, getters, setters, and overrides
}
```

Such tasks are perfectly suited to records as code readability is improved and the amount of code you need to write
is reduced. 

Using Records, this simplifies to: 

```java
public record Author(String firstName, String lastName){}
```

Records reduce boilerplate code by automatically generating constructors, `getters`, `equals()`, `hashCode()`, and `toString()` methods
based on the fields declared. Each field in a record is final, making records inherently immutable. 
This feature makes them ideal for use in situations where data consistency and thread safety are important,
such as in multi-threaded applications or when working with data transfer objects.


## Some key characteristics of Records

- A record can be top-level or nested, and it can be generic.
    ```java 
	public record Pair<T, U>(T first, U second) {}

	public class Main {
		public static void main(String... args) {
			Pair<String, Integer> stringIntegerPair = new Pair<>("Hello", 42);
			System.out.println(stringIntegerPair);
		}
	}
    ```
	`Pair` is a __generic top-level__ record that can hold two values of potentially different types.
 - Records are final and instantiated via the new keyword, and they cannot extend any other class.
 - Record class body may declare static methods, static fields, static initializers, constructors, instance methods, and nested types.
	```java 
	public record ExampleRecord(int id, String name) {
		 // Static field
    	private static final String CATEGORY = "Example";

    	// Static initializer
    	static {
        	System.out.println("Static initializer for ExampleRecord");
    	}

		//Nested class 
		public static class NestedClass {
			public void doSomething() {
				System.out.println("NestedClass method called")
			}
		}

		//Instance method
		public String getNameUpperCase() {
			return name.toUpperCase();
		}
	}
	```
	I appreciate that records in Java allow for inner classes, like the `NestedClass` static class example. 
	This is handy for adding specific functionalities to the record that don't align with its main purpose of holding data. 
	It ensures that the record's structure remains uncluttered while still offering this additional functionality.

 - Record classes can be nested. Because immediately enclosing instances can add states to the record, a nested record is implicitly static

 - Record classes can be serialized and deserialized. The serialization process is performed using record components, and the de-serialization process is performed using the canonical constructor
 - Java Record type definitions can also contain multiple constructors. 
 	_Below is an example of a Java Record that has an additional constructor._

	```java 
	public record Author(String firstName, String lastName){
		public Author(String firstName){
			this(firstName, null);
		}
	}
	```

Notice that the extra constructor calls the default constructor of the Author Record. 
Java's compiler requires this so that it will know which constructor parameters in the extra 
constructor correspond to which parameters in the default constructor.

Records can also be used for pattern matching. It's a new and exciting feature that is still being worked on, though some elements of this
feature have been released as final features. 
Look out my next post where I'll delve into this topic more -  it’s gonna be a hoot! Stay tuned and keep your coding socks on! 🤗🎉

This is being a very quick introduction of Records, more information about Records check the [docs](https://docs.oracle.com/en/java/javase/17/language/records.html) here.