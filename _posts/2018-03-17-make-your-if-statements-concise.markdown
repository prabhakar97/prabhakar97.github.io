---
layout: post
title: "Make your if statements concise"
date: 2018-03-17 20:17:27 +0530
comments: true
tags: [programming]
---
Sometimes you may encounter a situation where you need to do something if two conditions are satisfied or they both are not satisfied; and do something else
if exactly one of them is satisfied. An example would be:

```java
if ((resultFromServer1 == null && resultFromServer2 == null) || (resultFromServer1 != null && resultFromServer2 != null)) {
    log.warn("Both servers are aware or unaware of the operation");
    doSomething();
} else if ((resultFromServer1 == null && resultFromServer2 != null) || (resultFromServer1 != null && resultFromServer2 == null)) {
    log.err("Only one server is aware of the operation");
    doSomethingElse();
}
```

These conditions look messy. We can fix them using XOR! XOR returns true if two conditions are different and false if both conditions are same.
So we can write the above complexity as:

```java
if ((resultFromServer1 == null) ^ (resultFromServer2 == null)) {
    log.err("Only one server is aware of the operation");
    doSomethingElse();
} else {
    log.warn("Both servers are aware or unaware of the operation");
    doSomething();
}
```

The reverse of XOR is EX-NOR. And interesting thing about that is, in case of booleans EX-NOR is equivalent to both conditions being equal. So, it is also
called as equality gate. Hence, we can write:

```java
if ((resultFromServer1 == null) == (resultFromServer2 == null)) {
    log.warn("Both servers are aware or unaware of the operation");
    doSomething();
} else {
    log.err("Only one server is aware of the operation");
    doSomethingElse();
}
```
