+++
title = "The add and remove keywords in C#, event handlers"
date = 2024-08-16
updated = 2024-08-16
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

As you can see, we have a ’Player’ class, where we keep track of our points. The player can gain points (for instance, by collecting coins) and in the case they have at least 100 points (wow), they earn a shiny achivement.

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

I created this small example program as a Task-Based Async Patter (TAP), because that is the preffered way to add delays and realistically in a game these events should be nonblocking (other things must happen, like updating an enemy position).

Let's say I want to make a change in what happens when the achivement level is reached. In that case we would have to modify what happens in the `Player`'s source code. When we think about event and asynchronous execution or programming, this would be an anti-pattern. For instance many UI libraries would need to offer to rebuild the library which provides the functionality for this! This is very incovenient. And in the case of UI libraries or notification based systems, the caller should define what happens when an (event) completes, instead of the called object.

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
    private static void OnAchievementUnlocked(uint points)
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

**I will continue writing this post via an update, because it is a bit longer then excepted at first!** 
