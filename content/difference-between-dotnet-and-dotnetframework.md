+++
title = "What is the difference between .NET and .NET Framework? AKA Microsoft has a bad naming sense."
date = 2024-08-05
updated = 2024-08-05
description = ""
[taxonomies]
tags = ["C#","microsoft",".NET"]
+++

## Prologue

I do like working in C#. It is an elegant language, memory safe, it will run on any platform where the Common Language Runtime is available. It is so backwards compatible, that with some magic, you [could port any Windows app to Windows 95](https://youtu.be/CTUMNtKQLl8?si=XgFkojr5OULyg9Yx). Matt is crazy good that he managed to do this and the video is cinema production quality... so go and watch it.

Anyhow, dotnet versions exist, and that is a fact. Different version support different things. A .NET standard defines the APIs the framework provides. For each .NET version there are a set of languages from which their respective compiler can create a so called CIL, which stands for Common Intermediate Language.
