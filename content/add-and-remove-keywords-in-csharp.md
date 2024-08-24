+++
title = "The add and remove keywords in C#, event handlers"
date = 2024-08-16
updated = 2024-08-24
description = ""
[taxonomies]
tags = ["C#",".NET","programming"]
+++

When we talk about keywords, most people tend to think about context-independent keywords.
This is completely understandable as most of the time we tend to use those, like ’class’ or ’return’.
These are reserved keywords in C#. For all the types and possibilities in C# regarding keywords see [the official docs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/).

However in some cases there are certain words in our code, which count as keywords only in special situations.

To understand what `add` and `remove` is in C#, we must first dive a bit into events and delegates first. To do this we will look at a small program representing a game!

Let's say in our game the player can earn points. It does not really matter *how*, only that the player *can*.

```c#
// Player.cs
namespace EventHandlers {
    internal class Player {
        public uint Points {get; private set;}
        public async Task AddPoints(uint points) 
        {
            Points += points;
            Console.WriteLine($"Player earned {points} points!");
            await Task.Delay(1000);
            if(Points >= 100) Console.WriteLine("Congratulations! Achievement unlocked!");
        }
    }
}
```

As you can see, we have a ’Player’ class, where we keep track of our points. The player can gain points (for instance, by collecting coins) and in the case they have at least 100 points (wow), they earn a shiny achievement.

In our program we can instantiate the player and then add scores to it.

```c#
namespace EventHandlers;

public class Launcher
{
    private static async Task Main(string[] args)
    {
        var player = new Player();

        await player.AddPoints(50);
        await player.AddPoints(40);
        await player.AddPoints(40);
    }
}
```

I created this small example program as a Task-Based Async Patter (TAP), because that is the preferred way to add delays and realistically in a game these events should be nonblocking (other things must happen, like updating an enemy position).

Let's say I want to make a change in what happens when the achievement level is reached. In that case we would have to modify what happens in the `Player`'s source code. When we think about event and asynchronous execution or programming, this would be an anti-pattern. For instance many UI libraries would need to offer to rebuild the library which provides the functionality for this! This is very inconvenient. And in the case of UI libraries or notification based systems, the caller should define what happens when an (event) completes, instead of the called object.

One way to solve this problem is to introduce an event action in the `Player` class:

```c#
// Player.cs
namespace EventHandlers {
    internal class Player {
        private uint Points {get; set;}
        
        public event Action? AchievementUnlocked;
        public async Task AddPoints(uint points) 
        {
            Points += points;
            Console.WriteLine($"Player earned {points} points!");
            await Task.Delay(new Random().Next(500,1500));
            if(Points >= 100) 
            {
                AchievementUnlocked?.Invoke();
            }
        }
    }
}

```

The use for this is that when the caller calls our function `AddPoints(unit points)`, we will raise an event. The caller can decide to react to this.

```c#
namespace EventHandlers;

public class Launcher
{
    private static void OnAchievementUnlocked()
    {
        Console.WriteLine("Congratulations! Achievement unlocked!");
    }
    private static async Task Main(string[] args)
    {
        var player = new Player();
        
        player.AchievementUnlocked += OnAchievementUnlocked;

        await player.AddPoints(50);
        await player.AddPoints(40);
        await player.AddPoints(40);
    }
}
```

Now let's say, that we want to know *how many* points triggered this achievement notification. If we look at the [documentation](https://learn.microsoft.com/en-us/dotnet/api/System.Action?view=net-8.0) of the `Action?` class, we can see that is is described as:

> Encapsulates a method that has no parameters and does not return a value.

```c#
public delegate void Action();
```

As you can see this is actually a [delegate](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/) function.
The documentation mentions that a delegate is a type. If you ever written C or C++ code, there is a good chance that you have seen or written [function pointers](https://stackoverflow.com/a/840504/10653163). Delegates are similar to that, but encapsulate the methods to be called in a much safer way thanks to the language features provided by C#: object-orientation, type safety and memory safety.

As for the basics for working with delegates, there are 3 parts. One is for creating the delegate definition and the other is referencing and instantiating it. Then all we have to do is call the delegate. You can see a very simple example in the documentation, so I will continue refactoring our little game, the basic idea is the same.

```c#
// Player.cs
namespace EventHandlers {
    internal class Player {
        private uint Points {get; set;}
        public delegate void AchievementUnlockedHandler(uint points);
        public event AchievementUnlockedHandler? AchievementUnlocked;
        public async Task AddPoints(uint points) 
        {
            Points += points;
            Console.WriteLine($"Player earned {points} points!");
            await Task.Delay(new Random().Next(500,1500));
            if(Points >= 100) 
            {
                AchievementUnlocked?.Invoke(Points);
            }
        }
    }
}

```

```c#
namespace EventHandlers;

public class Launcher
{
    private static void OnAchievementUnlocked(uint points) // This is our event handler.
    {
        Console.WriteLine($"Congratulations! Achievement unlocked at {points} points!");
    }
    private static async Task Main(string[] args)
    {
        var player = new Player();
        
        player.AchievementUnlocked += OnAchievementUnlocked;

        await player.AddPoints(50);
        await player.AddPoints(40);
        await player.AddPoints(40);
    }
}
```

The great thing about event handlers, is that you can freely define and add more functionality to it. Let's say we have NPC traders. These traders get really impressed about the player after unlocking this specific achievement and want to trade with them.

You would probably implement this functionality in it's own class. Let's say we have the class `Trader`, where the trader NPC will want to sell the player something in function to how much points they have when unlocking the achievement:

```c#
// Trader.cs
namespace EventHandlers;

public class Trader
{
    public void TryTrade(int points)
    {
        System.Console.WriteLine($"Trader offers {points * 2} wood to the player.");
    }
}
```

We can add the callback as well:

```c#
//Launcher.cs
...
namespace EventHandlers;

public class Launcher
{
    private static void OnAchievementUnlocked(uint points)
    {
        Console.WriteLine($"Congratulations! Achievement unlocked at {points} points!");
    }
    private static async Task Main(string[] args)
    {
        var player = new Player();
        var trader = new Trader(); // Create an NPC trader.
        
        player.AchievementUnlocked += OnAchievementUnlocked;
        player.AchievementUnlocked += trader.TryTrade; // Call the trader's function.

        await player.AddPoints(50);
        await player.AddPoints(40);
        await player.AddPoints(40);
    }
}
...
```

In general we can see a pattern here. One object is defining and raising an event (publisher), while an other object listens to this raised event and reacts to it (subscriber).

It is very important to understand, that one should unsubscribe from the events they have subscribed to, because the garbage collector will not clean up objects if references are held to it. If the publisher object has a long lifetime and such a reference is held to it in the subscriber, then the subscription is such a reference.

This causes memory leaks, or rather unfreed memory to crop up over the lifetime of an application.

[Blazor](https://dotnet.microsoft.com/en-us/apps/aspnet/web-apps/blazor) is solves this by having component classes implement `IDisposable` and unsubscribing there:

```c#
@page "/example"

@implements IDisposable

<h3>Example Component</h3>

@code {
    private SomeService _someService;

    protected override void OnInitialized()
    {
        _someService = new SomeService();
        _someService.SomeEvent += OnSomeEvent;
    }

    private void OnSomeEvent(object sender, EventArgs e)
    {
        // Handle the event
    }

    public void Dispose()
    {
        // Unsubscribe from the event when the component is disposed
        if (_someService != null)
        {
            _someService.SomeEvent -= OnSomeEvent;
        }
    }
}
```

You can follow this example in your own code to clean up event handlers, as it is a fundamental part of .NET.

Of course there are situations, where it is not needed to unsubscribe manually. If the event handler is static (or as a special case of static, a global one), you might not need to do so, because the number of references are fixed and the lifetime is known. Also, even if the number of subscriptions are well defined, if you know for a fact the specific publisher instance is shorter lived than the subscriber, the garbage collector can clear up the associated memory, as when the subscriber goes out of scope, the event handler will no longer point to an in-scope publisher.

Design patterns can help you achieve this. Using the **Dispose pattern** or **IDisposable pattern** help you in correctly implementing resource cleanup for your classes. This is very useful, because as C# is a managed language and the CLR does provide a really good way to forget about managing resources manually, beyond a certain abstraction, it must be cared for, like unmanaged resources (file handlers, database connections) and event handlers. (Python is actually more advanced in this regard, because via the `with:` block, you can make sure resources are cleaned up and no exception handling is needed.)
Implementing these patterns usually refers to the same thing, they just accentuate different aspects of the same idea.

.NET also provides a great addition so we do not have to manually define the delegate function.

You could use `EventHandler<T>` type in the event definition as the type, where `T` is the event data:

```c#
//EventHandlers.cs
using System;
using System.Threading.Tasks;

namespace EventHandlers {
    internal class Player {
        private uint Points { get; set; }

        // Define the event using EventHandler<T>
        public event EventHandler<uint>? AchievementUnlocked;

        public async Task AddPoints(uint points) 
        {
            Points += points;
            Console.WriteLine($"Player earned {points} points!");
            await Task.Delay(new Random().Next(500, 1500));
            if (Points >= 100) 
            {
                // Raise the event, passing the points as event data
                AchievementUnlocked?.Invoke(this, Points);
            }
        }
    }
}
```

If we had multiple parameters in our delegate function, you could create a custom class which encapsulates all the event data and has a constructor, which can be accessed in the given context and assign that custom type to `T`.

Also, you do not need to define any delegate type, when using `EventHandler`, because the default delegate function signature for it is `object? sender, EventArgs e` with a `void` return type.

It should be noted that when you define your own delegate with no arguments and make an event with that type, it will no longer match `object? sender, EventArgs e`, but rather it will have a no parameter type.

So in the most general sense, every event must have a delegate type, but .NET provides multiple predefined classes with good patterns. I would recommend using `EventHandler<T>` when possible, because you certainly should care about the sender and the event arguments. In most cases events also are only side-effect producing functions and you do not want to return from them.

`Action<T>` is a more free-form way of defining the event type, because `object? sender, EventArgs e` is not required, but this also means that you kind of break the standard expectation. I would only use this if there is a significant case of speed-up and only internally if possible.

**IMPORTANT!**
When you define your own delegate, you can also define a return type. But adding this to an action is an anti-pattern, not only because it goes against standard .NET conventions (and many libraries use them, especially `EventHandler` for event-driven programming), but also because in my opinion it breaks the single responsibility from SOLID, because event handlers are not for data retrieval on the type level. The subscriber might write their own retrieval of the data, provided there is a getter for it or is available in the scope.

Now after all of this, we can understand finally what `add` and `remove` are.

When we type the addition and removal operators (`+=` and `-=`) to add or remove event handlers, we actually call the compiler generated `add` and `remove` methods.

Similar to C++ operator overloading, you might want to define how it is done. For instance, you could have a debug build, where it is logged or you might want to optimize performance, by calculating subscription metrics and a lot more.

For simplicity's sake, I will show a new example. Let's say we have this very simple class with the default `EventHandler`:

```c#
public class MyClass
{
    // Declare an event using the default add and remove behavior
    public event EventHandler MyEvent;

    public void RaiseEvent()
    {
        MyEvent?.Invoke(this, EventArgs.Empty);
    }
}
```

Somewhere else it is called like this:

```c#
var obj = new MyClass();

// Subscribe to the event
obj.MyEvent += EventHandlerMethod;

// Unsubscribe from the event
obj.MyEvent -= EventHandlerMethod;

void EventHandlerMethod(object sender, EventArgs e)
{
    Console.WriteLine("Event triggered!");
}
```

This code translates mostly into:

```c#
public class MyClass
{
    // Underlying delegate field
    private EventHandler? _myEvent;

    // Custom add and remove accessors (generated by the compiler)
    public event EventHandler MyEvent
    {
        add
        {
            // Adds a handler to the event
            _myEvent += value;
        }
        remove
        {
            // Removes a handler from the event
            _myEvent -= value;
        }
    }

    public void RaiseEvent()
    {
        // Invoke the event handlers
        _myEvent?.Invoke(this, EventArgs.Empty);
    }
}
```

This would still hold true for any combination or actual implementation of `EventHandler<T>`, `Action<T...>` or your own delegate.

Of course, when we use `Action` or `EventHandler`, we do not need to define the delegate field when using `add` or `remove`:

As an example of a real-world use case I use [Microsoft's documentation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/add):

```c#
class Events : IDrawingObject
{
    event EventHandler PreDrawEvent;

    event EventHandler IDrawingObject.OnDraw
    {
        add => PreDrawEvent += value;
        remove => PreDrawEvent -= value;
    }
}
```

Here, the we have a class `Events`, which implements the `IDrawingObject` interface. In the `IDrawingObject` there is method defined `OnDraw`. This class very well demonstrates how one can use custom event accessors `add` and `remove` to redirect the event to our own defined event handler, which is `PreDrawEvent`. This is actually a common pattern for handling events defined in interfaces.

So all in all, we see that even simple keywords might need a lot of background knowledge to comprehend. Events are an essential part of programming, especially UI related programming, which C# is heavily used for. I hope I could help you all to better understand these concepts.
