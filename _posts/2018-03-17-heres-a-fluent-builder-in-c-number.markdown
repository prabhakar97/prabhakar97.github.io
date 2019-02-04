---
layout: post
title: "Here's a fluent builder in C#"
date: 2018-03-17 20:01:34 +0530
comments: true
tags: [c-sharp, design-patterns]
---

A fluent builder makes creating objects more readable than constructors and more concise than setting properties through setters
on an uninitialized object.
Java devs can use Lombok's `@Builder` annotation to get a fluent builder wired into their classes. Unfortunately, in C# world
no such thing exists. So here's how you can roll out your own.

```c#
public class Animal
{
  public string Name { get; set; }

  public string Type { get; set; }


  #region Fluent builder
  public static Animal Builder()
  {
    return new Animal();
  }

  public Animal WithName(string name)
  {
    this.Name = name;
    return this;
  }

  public Animal WithType(string type)
  {
    this.Type = type;
    return this;
  }

  public Animal Build()
  {
    return this;
  }
  #endregion
}
```

And, to construct and object you can simply use:

```c#
Animal a = Animal.Builder().WithName("Tommy").WithType("Dog").Build();
```

## Update

C# doesn't need this. I figured it out after reading and writing lot of C# code. C# already has a nice intializer which can be used instead
of a fluent builder:

```c#
Animal a = new Animal
{
    Name = "Tommy",
    Type = "Dog"
};
```
