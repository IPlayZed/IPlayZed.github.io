+++
title = "Extension methods in C#"
date = 2024-08-27
updated = 2024-08-28
description = ""
[taxonomies]
tags = ["C#","microsoft",".NET", "extension methods"]
+++

***This post is a WIP.***👷‍♂️🏗️

We already looked at a post explaining events and delegates that libraries might want to provide us with functionality to decide what happens at certain points in our application, especially if these libraries are GUI libraries. This was event driven programming.

However, not all use cases are meant to be done via event driver programming and there definitely are cases where we would want to extend the functionality of a library, without recompiling it, either because we can't (no source code) or don't want to (too complicated). Traditionally you could just write the relevant wrappers to it, but C# provides advanced syntactic sugar to call specific kind of methods as if they were the part of the library.

In this case C# provides extension methods. It is a similar concept to the decorator design pattern, but that is a rather dynamic way of extending the functionality via common interfaces, but extension methods are completely compile time structures and have specific rules about them.
