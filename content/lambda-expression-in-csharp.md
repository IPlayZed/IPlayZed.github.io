+++
title = "Lambda expressions in C#"
date = 2024-08-28
updated = 2024-08-28
description = ""
[taxonomies]
tags = ["C#","microsoft",".NET", "lambda expressions"]
+++

***This post is a WIP.***👷‍♂️🏗️

In many cases there is a fine blur between adding language features for functionality and syntactic sugar.
At the very base level lambdas are "just" a way to define small function, without th need to name them, at the place where they are used. However, taking a closer look at them shows us, that they are really closely related to the idea of delegates and due to that they can make the code easily readable, for instance when using LINQ.

Let's say that we are in the times before C# 3.0. The Great Recession in getting ready to hit the global economy harder than ever before and you wish to write a function to add two numbers together for you Windows XP app. You would do something like this:

```c#
namespace lambdas;

public delegate int MyOperation(int x, int y);

public static MyApp
{
    public static void Main()
    {
        MyOperation operation = delegate (int x, int y)
        {
            return x + y;
        }

        int result = operation(3, 4);
        Console.WriteLine(result) // Output: 7
    }
}
```

This is great, because we now are able to define many unnamed operations, which correspond to that delegate.
However, this is not very programmer friendly, because we still need to declare the delegate, define it, call it...

Since C# 3.0, you can use lambdas instead. This introduces the arrow operator, written as `=>`.

If we really did not want to write up a delegate, maybe we could just use a local function. This could be good, because local functions can be called in many different places and they can only be accessed from the member they are declared in. Also, we do not have to name it a lot differently:

```c#
namespace lambdas;

public static MyApp
{
    public static void Main()
    {
        // Call the delegate
        int result = Operation(3, 4);
        Console.WriteLine(result); // Output: 7
        return;

        // Create a delegate instance using an anonymous method
        int Operation(int x, int y)
        {
            return x + y;
        }
    }
}
```

Local function, however are only available since C# 7.0 or later (since 2017, a whole 10 years of advancements). If you require recursion, but still want to make functions local-only, then it is the only way you can go for now. Same goes for named parameters and there probably are some other edge cases. However, for the most part you have lambdas and do not forget, that while some features might not be supported by them, the compiler can assign them to delegates, so anything you can work the general `Function` and `Action` delegates, you can do with lambdas. Also, lambdas, while have add a good chunk of functionality are also a syntactical thing for the most part.
