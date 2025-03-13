---
layout: post
title:  "The Latest Features in Java 21 - Sequenced Collection"
date:   2024-03-20 
categories: java update
---

### TL;DR 🚀 
Java 21 introduced sequenced collections, a new framework that unifies and standardizes operations for ordered collections. This feature adds interfaces like `SequencedCollection`, `SequencedSet`, and `SequencedMap` that provide first/last element access, reverse views, and bidirectional iteration. It enhances existing collection classes like `ArrayList`, `LinkedHashSet`, and `TreeMap` without requiring code rewrites, making it easier to work with ordered data structures in a consistent way.


### Motivation
Java's `Collection` framework does not include a `Collection` type that signifies a sequence of elements with a specified order of encounters. 
it is also missing a uniform set of operations applicable across such `Collections`, which is often frustrating.

For instance, collections like `List` and `Deque` maintain a specific order of elements, akin to a sequence. However, 
their overarching category, `Collection`, does not enforce any order. This is similar to `Set`, 
which also does not require a specific arrangement, with certain types like `HashSet` following suit. Yet,
some subsets of `Set`, such as `SortedSet` and `LinkedHashSet`, do adhere to a defined order. 
This inconsistency in enforcing order across the hierarchy complicates the expression of certain concepts 
in the API. As a result, specifying a parameter or return type that preserves order becomes challenging: 
`Collection` is too broad, while `List` is too narrow, excluding types like `SortedSet` and `LinkedHashSet`
that do maintain an order.

A similar issue arises when collections that support a specific order, like `LinkedHashSet`, 
are wrapped in a way that restricts their functionality, take a look the following code snippet. 
```java 
    LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>();
    linkedHashSet.add("Apple");
    linkedHashSet.add("Banana");
    linkedHashSet.add("Cherry");

     // Wrap the LinkedHashSet in an unmodifiable set
    Set<String> unmodifiableSet = Collections.unmodifiableSet(linkedHashSet);
```
The `LinkedHashSet` is wrapped with `Collections.unmodifiableSet`, making it immutable. 
While the original order of elements is maintained, the wrapped set doesn't allow any modifications, and the specific qualities
of `LinkedHashSet` (like its ordered nature) are not directly accessible or evident through the unmodifiable set interface.
This action strips away the order information, reducing the collection to a basic `Set` with no guaranteed order.

Furthermore, without specific interfaces to handle order, methods to access ordered elements,
like the first or last item, are not standardized. Different collections might offer these features, but 
they do so in their own unique ways, leading to inconsistency.
In some cases, these functionalities might not be available at all.
Here is an example: 
```js 
|                | Retrieving first element      | Retrieving last element      |
|----------------| ---------------------------   | ---------------------------- |
| List           | list.get(0)                   | list.get(list.size()-1)      |
| LinkedHashSet  | linkedHashSet.iterator().next | //doesn't exist              |
| SortedSet      | sortedSet.first()             | sortedSet.last()             |
| Deque          | deque.getFirst()              | deque.getLast()              |
```
Some operations, such as obtaining the last element of a List, are unnecessarily complex, and others, 
like retrieving the last element of a `LinkedHashSet`, require full iteration through the `Set` - a process that is not straightforward. 
These limitations have consistently led to technical challenges.

### The SequencedCollection Interface to the rescue

The Sequenced Collections API introduces new interfaces to the existing Collection Hierarchy to cater to 
collections that maintain a specific order of elements.
Many of the methods in this new API are not entirely new; 
they are existing methods that have been elevated from more specific classes to these broader interfaces.

On the collections side two new interfaces were added, `SequencedCollection` which contains the following methods 
```java 
 interface SequencedCollection<E> extends Collection<E> {
    // new method
    SequencedCollection<E> reversed();
    // methods promoted from Deque
    void addFirst(E);
    void addLast(E);
    E getFirst();
    E getLast();
    E removeFirst();
    E removeLast();
}
```

and `SequencedSet` which extends `SequencedCollection` and overrides its `reversed()` method. 
The only difference is the return type of `SequencedSet.reversed()` which is `SequencedSet`.

```java
interface SequencedSet<E> extends Set<E>, SequencedCollection<E> {
    SequencedSet<E> reversed();    // covariant override
}
```
#### SequencedMap 

On the Map side of the Collections Hierarchy a single interface was added `SequencedMap`. 
A sequenced map is a type of Map where the entries follow a specific order. Unlike a `SequencedCollection`, 
the `SequencedMap` stands alone and offers unique methods for adding or removing elements from the start or end of the map.

_Let's take a closer look at the contents of the `SequencedMap` interface_
```java 
interface SequencedMap<K,V> extends Map<K,V> {
    // new methods
    SequencedMap<K,V> reversed();
    SequencedSet<K> sequencedKeySet();
    SequencedCollection<V> sequencedValues();
    SequencedSet<Entry<K,V>> sequencedEntrySet();
    V putFirst(K, V);
    V putLast(K, V);
    // methods promoted from NavigableMap
    Entry<K, V> firstEntry();
    Entry<K, V> lastEntry();
    Entry<K, V> pollFirstEntry();
    Entry<K, V> pollLastEntry();
}
```

The Sequenced Collections API enhances collections that maintain a specific sequence of elements, 
impacting many widely-used collections such as ArrayList and SortedSet. 
However, collections like HashSet, which don't have a fixed order of elements, are not affected by these updates.

In the realm of maps, commonly used `HashMaps` don't benefit from the 
Sequenced Collections API enhancements due to their lack of a defined element sequence.
To leverage the new features introduced in the `SequencedMap`,
you would need to opt for map implementations that maintain order, such as `TreeMap` or `LinkedHashMap`.

_Something that looks like this_ 
    ```java 
        SequencedMap<Integer, Integer> map = new LinkedHashMap<>();
        map.putIfAbsent(0, 34);
        map.firstEntry(); // Returns the first key-value mapping in this map, or null if the map is empty.
    ```

#### Conclution
In summary, the introduction of Sequenced Collections represents a major advancement 
for Java Collections, effectively meeting the demand for a consistent approach to managing ordered collections.
This enhancement allows developers to operate more seamlessly and logically. 
The addition of new interfaces brings about a more coherent framework and uniform functionality, leading to stronger and more understandable code.