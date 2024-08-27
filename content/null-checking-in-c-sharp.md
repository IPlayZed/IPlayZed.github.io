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
