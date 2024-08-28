+++
title = "Checking for null values in .NET"
date = 2024-08-27
updated = 2024-08-27
description = ""
[taxonomies]
tags = ["C#","microsoft",".NET", "null checking", "type safety", "nameof operator"]
+++

By the design of C# reference types can have the value of `null`. This is perfectly normal and developers are supposed to use it as a feature. The problem stems from most developers not taking care of it, when working with references (which is on the most part of the job). As when we call on a member of a `null` object reference, it causes `NullReferenceException`, which is probably not managed on the application level causing a crash.

While the most common causes of crash in invalid memory access in C and C++, the main cause of crashes is probably `null` dereference.
To do this, the easiest way is to check the object reference if it is equal to `null` via the `==` equality operator. For `String` types, I recommend using the `IsNullOrEmpty()` method, as empty string probably mean the same as `null` in the application context.

Since C# 6.0 you can use the unary and ternary null propagation operators to simplify checking for `null`.
Let's say you have the object `nullableObject`, which can have a `null` value. If you write `nullableObject?.myValue` or `nullableObject?.myMethod()`, it conditionally only calls the part after the dot, if the object was not `null`.

You can also use `??` like `nullableObject?.myValue ?? otherValue` (and similarly for method calls) which produces either the left hand side `myValue` if `nullableObject` is not `null` or `otherValue` if it is (or the method calls' return value).

On this note, it is a good practice, if you want to explicitly handle if a reference was `null` and output an exception about it, I would use the `nameof(myObject)`, in an exception call, for instance:

```c#
void myMethod(string s)
{  
   if (s == null) 
   {
      throw new ArgumentNullException(s);
   }  
}
```

This is unreliable, because there is no guarantee, that the exception message will correctly resolve for the variable name, especially if `s` was `null`.

Also, you could pass `"s"` as the exception message, but the problem with that is that if the actual parameter name is changed, it might be that the exception message is not modified by accident, leading to confusion, but at the very least unneeded maintenance costs.

If you however do:

```c#
void myMethod(string s)
{  
   if (s == null) 
   {
      throw new ArgumentNullException(nameof(s));
   }  
}
```

The compiler will always resolve to the actual parameter name, making debugging easier.

It is also a good idea to clear up the difference between nullable and non-nullable reference types.
Anything that is a reference and it's type is marked with a postfix `?` is meant to be nullable. This was introduced in C# 2.0, so the concept is very standard, but it was only valid for value types, like `int` and nullable reference types are only available since C# 8.0.

This language feature is useful because when you try to assign a `null` value to a non-nullable type, the compiler will generate warnings. For instance you have the following code, by default since C# 8.0 these would generate a warning:

```c#
string notNullableString = null; // You can not assign null to a non-nullable reference.

string? nullableString = null // Create a nullable reference.
Console.WriteLine(nullableString.Length) // It is nullable, and it was dereferenced before checking it, throws a null exception.
```

This also means that if you did not mark a reference as nullable, you can not dereference it in any way before initializing it.

However, this does not generate a warning:

```c#
string myStr? = null; // You can assign null to it, as it was explicitly declared.
Console.WriteLine(myStr?.Length) // This will evaluate to null, which can be written out and does not throw a null exception.
```

This is good, because the caller understands that `myStr` is nullable as it is explicitly marked and the compiler will check assignments and dereferences of it.

Of course for pretty programs you would want to define alternative behaviors to the path if it is `null`. I would strongly advise to use the `??` operator if you have some simple logic.

In practice, this happens a lot when working with EF (Entity Framework, and by EF I mean both Core and not Core), so in the context of databases. This means that you would need a lot of repetition to handle `null` values at the dereferencing point. This is not very good because you have to repeat yourself. (I know that D stands for dependency inversion in SOLID, but I would call this extra hardening, because in my ideal world SOLID also contains the famous *don't repeat yourself* mantra, so if you follow all the other SOLID principles, you might still have to `null` check everywhere... But I am happy for anyone to tell me that I misinterpret something and one of the principles actually would dictate what I am about to show you.)

I think it should be explicitly shown that for a given type you are dereferencing you want to get the value or a default type. Let's say you have a class `MyClass`, you could create an extension method for getting the value or default, so no repetition in needed:

```c#
// Define an example class with one member.
public class MyClass
{
   public string Name { get; set; }
}

// Define extension methods for MyClass
public static class Extensions
{
   public static MyClass OrDefault(this MyClass instance)
   {
      return instance ?? new MyClass { Name = "Default name" };
   }
}

// Demonstrate usage
class Program
{
    static void Main()
    {
        MyClass instance = null;
        MyClass result = instance.OrDefault();

        Console.WriteLine(result.Name);  // Output: Default Name
    }
}
```

This is best for small objects. For other objects this is not extremely efficient as it allocates a new class instance. You might want to look into factory patterns or similar for that.

It should be noted that this behavior can be easily overridden, either in the project configuration or in code using preprocessor commands. You can disable it completely, configure it so it is only a syntactical feature, set it so warnings are generated by the compiler (the default behavior) and set it so errors are generated by the compiler. The safest and the strictest solution is generating build error, but in my opinion you should set it to this, because it would cause an exception anyway, which would need to be handled.

If you want to configure it on the project level, you need to set it in the `.csproj` project configuration file.
Under the `<PropertyGroup>`, you need to set `<Nullable>` to `enable`. You also have to set `<TreatWarningsAsErrors>` to `true`.
For setting it to only be an annotation (no warnings even), set `<Nullable>` to `annotations`. To disable it from the language (treat it as pre C# 8.0, which I would only do if you want to keep legacy logic, so I do not recommend it), keep it `<Nullable>` to `disable`.

If you want fine grained control over when it is used (better for migration from legacy code, as it is explicit in my opinion), you can use preprocessor commands:

```c#
#nullable enable warnings
// Here, nullable reference types are enabled, and warnings will be generated.

string? nullableString = null;

#nullable enable annotations
// Only the annotations part of the nullable feature is enabled. This will not generate warnings but allows you to use nullable reference annotations (e.g., `string?`).

#nullable disable
// Nullable reference types are disabled, and no warnings will be generated.

string nonNullableString = null; // No warning, even though it should be non-nullable.

#nullable restore
// Restores the nullable context to the project default.
```

Of course, there are further ways you can handle it. Personally I find Rust's way of handling it ingenious, as it is a natural way, which arises from the language itself:

```rs
enum Option<T> {
    Some(T),
    None,
}
```

They have an `enum`, which implicitly captures the notion if a missing value and an existing value for any type.

Similar things can be implemented and used in C#. While the language is not meant to work like that, I still am convinced that this is the most natural way, as it forces explicit checking and arises from the type system.
