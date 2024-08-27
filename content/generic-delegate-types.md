+++
title = "Generic delegate types in .NET"
date = 2024-08-25
updated = 2024-08-25
description = ""
[taxonomies]
tags = ["C#","microsoft",".NET", "generic", "delegate"]
+++

***This post is a WIP, but I posted it because it already contains some cool information***😁

Recently I have written a blogpost about the `add` and `remove` keywords. We had to dvelve into multiple concept, including delegates. For a quick refresher: delegates are similar to function pointers from the world of C/C++. They are object for storing information about the signature of a method, making it possible to to later define or call a method on an object matching their signature. This is useful for defining for instance what kind of reactions can be provided for events.

Generics are used to introduce the concept of type parameters. The primary use of them is to be able to define fully functional classes and methods that defer the specification of one or more type parameters until that given class or method is used somewhere else in the code. Generic classes are meant for reusability, type safety and efficiency. When compilation happens, generic type parameters are replaced by the caller's argument(s). (Parameter: the symbol used to mark the symbol value in a signature. Argument: the actual value of the symbol. This is true for any function, even in math.)

One of the most common use cases of generics is in containers, which we will have a lengthy look at in a later blogpost. The reason for it is that containers are meant to be used on any type of data, and their operations are clearly defined to be independent from the data they store. (Of course for custom solutions, this might not hold true.) You can find many in the `System.Collections.Generic` namespace in .NET.

A cool feature of generics is that they provide a way to make constraints regarding the type parameters. The reason why this is not a limitation, is that when creating a generic, you can introduce operations on the generic parameters, which are supported by the constrained type. You can also make a constraint where you define what something is not, allowing you to make operations, otherwise not available. By default all generic parameters unless constrained otherwise are `System.Object` type, meaning that you can only do operations, which are supported by that. It is a very broad topic, so I would recommend taking a look at [the programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters) to see all the possible ways you can make them and it can get especially messy, because some versions of C# might not allow constraints, which are otherwise allowed in the CLR.

Generics are similar to templates in C++. Emphasis on similar, because it is clearly defined that they do not wish to implement the same feature set or syntax that templates do in C++. The very core idea of templates in C++ is that it is a tool to programmatically define our program. In other words, it is a way to write down the rules, which produce program code, so we program the compiler to produce code. It is completely compile-time. This is why C++ template programming is also called metaprogramming, because we work on a level higher, than our program code will.

Briefly, I would like to show what is *not* provided in generics, but is in templates:

- Type parameters can not have default types.

There are 5 main groups of delegates. They can be categorized according to return and functionality:

There is the concept of variance in generics that is needed to fully understand and distinguish between generic delegate types.
Variance allows the programmer to define and use more derived types (covariance) or more generic types (contravariance). There are 2 contextual keywords associated with this: `out` is for covariance and `in` is for contravariance. Both keywords apply to a generic type so they must be used like `<out T>` or `<in T>`, where `T` is the generic type parameter.

Let's see how covariance, the `out` keyword works:

Let's say I have a program for managing a pet shop. As a base class, I define animals, but because I also work with dogs and they have their own needs, I define a class for dogs:

```c#
public class Animal {}
public class Dog : Animal {}
```

Now, we probably have to order some animals if we want to sell them. As a general employee responsibility, for any type of animal this can be defined, so we define this as an interface. We also specify this for dogs, so we create a `DogOrder` class, which implements the `IOrder` interface for the `Dog` class, thus we give it the type parameter `IOrder<Dog>`.

```c#
public interface IOrder<out T>
{
    T Order();
}
public class DogOrder : IOrder<Dog>
{
    public Dog Order()
    {
        return new Dog();
    }
}
```

Now because we defined `T` as `out T` in `IOrder<T>`, it means that as `T` is the return type, the returned object can be assigned to the parent class, and this by definition is the meaning of covariance:

```c#
class Program
{
    static void Main()
    {
        IOrder<Dog> dogOrder = new DogOrder();
        IOrder<Animal> animalOrder = dogOrder; // Covariance allows this assignment

        Animal animal = animalOrder.GetItem();
    }
}
```

In the context of the example this might mean something like that the pet shop employee may fill out a form for a dog order, but then copy all the info to a generic pet shop order and make the order, because the supplier will know how to handle it.

Due to the generalization feature of covariance, it might only be used as return types of methods.

### Action-type generic delegates

We have already seen these in my blogpost about `add` and `remove`.

```c#
public delegate void Action();
public delegate void Action<in T1,in T2>(T1 arg1, T2 arg2);
public delegate void Action<in T1,in T2,in T3>(T1 arg1, T2 arg2, T3 arg3);
...
public delegate void ACtion <in T1,...,in T16>(T1 arg1, ..., T16 arg16);
```

These are all void type delegates, which support from no input up 16 any type of object input. They are mostly meant for event handling, but the `EventHandler` is preferred to it in that context. I would use it for any purely side-effect producing function, which is not strictly an event. Also, I think that for logically coherent parameters or more than a few, like 3, I would encapsulate them as their own object and pass that as an argument.
