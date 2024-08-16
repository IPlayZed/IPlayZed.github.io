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

## Event handlers

To understand what `add` and `remove` is in C#, we must first dive a bit into event handlers first. To do this we will look at a small program representing a game!

Let's say in our game the player can earn points. It does not really matter *how*, only that the player *can*.

```c#
namespace EventHandlers {
    internal class Player {
        public uint Points {get; private set;}
        public async Task AddPoints(uint points) {
            Points += points;
            Console.WriteLine($"Player earned {points} points!");
            await Task.Delay(1000);
            if(Points >= 100) Console.WriteLine("Congratulations! Achievement unlocked!");
        }
    }
}
```

As you can see, we have a ’Player’ class, where we keep track of our points. The player can gain points (for instance, by collecting coins) and in the case they have at least 100 points (wow), they earn a shiny achivement.

In our program we can instantiate the player.


