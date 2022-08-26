+++
title = "Using abstract class and interface in Java"
description = "A set of conditions to determine when to use an abstract class and when to use interface in Java"
date = 2021-12-12T11:55:24+05:30
featured = true
draft = false
comment = true
toc = true
reward = true
diagram = true
categories = [
  "Programming"
]
tags = [
  "java",
  "best-practice"
]
series = [
  "Manual"
]
+++

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
The term `behavioural siganture` in this article essentially means, a method signature in an interface or a class.
The classes which implement those behaviours can have completely independent and unrelated implementation details,
as needed.
{{< /note >}}

## Usecases

__Entities of same kind, requiring a common set of behavioural signatures.__

The requirement here is to have multiple entities of the same kind, all of which requiring a common set of
behavioural signatures.

Solution:

1. Implement an `abstract base class` with abstract and non-abstract methods. The non-abstract methods are meant
   to provide base implementation so that, the derived entity classes can skip their implementations as needed.
2. Extend the abstract class with appropriate entity classes, overriding the methods as required.

{{< tip title="SAMPLE" >}}
Checkout the sample Java classes - `Shape.java`, `Circle.java`, `Rectangle.java` under
[class-vs-iface](https://github.com/ravihara/java-points/tree/main/class-vs-iface). Here, the 'Circle' and 'Rectangle'
classes are of the same kind 'Shape'. Hence, they extend the 'Shape' base class.
{{< /tip >}}

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

{{< tip title="SAMPLE" >}}
Checkout the sample Java classes - `Authenticator.java`, `User.java`, `AppClient.java` under
[class-vs-iface](https://github.com/ravihara/java-points/tree/main/class-vs-iface). Here, 'User' and 'AppClient' classes
are of different kind. 'User' class represents a person or, user while 'AppClient' represents a registered, application-client
for a given web service. Hence, both these classes implement the interface 'Authenticator' independently.
{{< /tip >}}

__Entities of mixed kinds, requiring a common set of behavioural signatures.__

The requirement here is to have multiple, closely related and unrelated entities, all of which requiring
a common set of behavioural signatures.

Solution:

1. Define an `interface` covering all the method signatures, as required by the entity classes.
2. Define various `abstract classes` implmenting the defined interface, each for a set of closely related entities.
3. Implement the defined interface directly for entity classes which do not derive from any of the
   abstract class(es) as defined above.
4. Create various closely related entity classes by extending the appropriate abstract class. Each of them may
   further override the methods as appropriate.
5. In the future, if any new method signature needs to be added to the interface, do provide a default
   implementation so that existing classes (abstract or otherwise) still satisfy the interface definition.
   This allows the implementing classes to provide overridden method implementations in the future, as required.

{{< tip title="SAMPLE" >}}
Checkout the sample Java classes - `Authenticator.java`, `User.java`, `LeadUser.java`, `SalesUser.java`,
`AppClient.java` under [class-vs-iface](https://github.com/ravihara/java-points/tree/main/class-vs-iface). Here,
the classes 'User' and 'AppClient' are of different kind and hence, implement the 'Authenticator' interface independently.
The classes 'LeadUser' and 'SalesUser' are of the kind 'User'. Hence, they extend the 'User' which, also
satisfies the 'Authenticator' interface.
{{< /tip >}}

The maven based, sample Java project explaining the above usecases can be found in the
[class-vs-iface](https://github.com/ravihara/java-points/tree/main/class-vs-iface) module folder of the
[java-points](https://github.com/ravihara/java-points) git repository;
