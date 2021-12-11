+++
title = "Useful points for Java coding"
description = "A set of useful points which I learnt while coding in Java"
date = 2021-12-11T11:55:24+05:30
featured = true
draft = true
comment = true
toc = true
reward = true
categories = [
  "programming"
]
tags = [
  "java",
  "coding",
  "best-practice"
]
series = [
  "Manual"
]
images = []
+++

<!--more-->

This post captures various best-practices and/or useful suggestions which might come handy while programming in Java.
These are purely my observations and opinion using Java as the programming language to create backend services and
cross-platform applications. Moreover, I have been using the latest stable version of Java (v17 LTS). Hence, some of
the points might not be relevant to older versions. But, I try to capture the features which are common enough in the
recent past.

One of the fundamental mental model that a typical Java developer needs to develop is, to get a clarify
on when to use a Class and when to use an Interface. In fact Class and Interface, if used judiciously,
can compliment each other and adapt natually to suite one's programming goals. Instead of being too descriptive,
let me jot down some of the usecases and try to explain those with an example.

## A quick recap of OOP with Java

1. In an OOP language such as Java, an object represents an entity which has states and behaviours.
2. The state is typically captured via the fields / member variables of the object.
3. The behaviour is typically handled via the methods / member functions of the object.
4. The state should be internal to an object. It should not be exposed and modified directly by externally.
5. The behaviours are meant to handle all the state changes and act upon internal events (within the object) and
   external events (within the system in which the object belongs and also from the connected ecosystem).

{{< note title="NOTE" >}}
The term `behavioural siganture` in this article essentially means, a method signature in an interface. The classes
which implement those behaviours can have completely independent and unrelated implementation details, if needed.
{{< /note >}}

## Usecases

__Entities of same kind, requiring a common set of behavioural signatures.__

The requirement here is to have multiple entities of the same kind, all of which requiring a common set of
behavioural signatures.

Solution:

1. Implement an `abstract base class` with abstract and non-abstract methods. The non-abstract methods are meant
   to provide base implementation so that, the derived entity classes can skip their implementations as needed.
2. Extend the abstract class with appropriate entity classes, override and implement the abstract methods
   of the abstract, base class.

__Entities of different kind, requiring a common set of behavioural signatures.__

The requirement here is to have multiple, unrelated entities, all of which requiring a common set of
behavioural signatures.

Solution:

1. Define an `interface` covering all the method signatures, as required by the entity classes.
2. Provide default implementations of methods if some of the entity classes might not implement one or
   more interface methods immediately.
3. Create various entity classes implementing the interface.
4. In the future, if any new method signature needs to be added to the interface, do provide a default
   implementation so that existing entity classes still satisfy the interface definition. This allows
   the entity classes to provide overridden method implementations in the future, as required.

__Entities of mixed kinds, requiring a common set of behavioural signatures.__

The requirement here is to have multiple, closely related and unrelated entities, all of which requiring
a common set of behavioural signatures.

Solution:

1. Define an `interface` covering all the method signatures, as required by the entity classes.
2. Define various `abstract classes` implmenting the defined interface, each for a set of closely related entities.
3. Implement the defined interface directly for entity classes which do not derive from any of the
   abstract class(es) as defined above.
4. Create various closely related entity classes by extending the appropriate abstract class.
5. In the future, if any new method signature needs to be added to the interface, do provide a default
   implementation so that existing classes (abstract or otherwise) still satisfy the interface definition.
   This allows the implementing classes to provide overridden method implementations in the future, as required.
