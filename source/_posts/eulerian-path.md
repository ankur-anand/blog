---
title: Self Note on Eulerian Path
date: 2016-04-21 21:21:19
tags:
- Python
- graph
category:
- Algorithms
---

### Introduction

Eulerian Path are paths that
1. Start at some node
2. Visit every node exactly once
3. And ends

![Eulerian paths](https://raw.githubusercontent.com/ankuanand/Blogs-Image/master/eulerian1.png)

For the above Diagram We can start at some node for example say
We starts at node **D**
The next criteria is Visit every node exactly once

In the above graph we can transverse in any manner. I'm transversing in following manner.
>D -> B -> A -> D -> C -> A

So in the above we have an Eulerian path that started at D and ended at A

### Condition

>If a graph is connected and have a two node with even degree, than it has an
>Eulerian path.

As seen from above example the starting and ending node D and A has an odd degree.

If Graph has all odd degree that graph can't have a Eulerian Path

### Exception

>If all the node is of even degree

For example
The below graph has an Eulerian Path even when all of it's node is even

![Even Eulerian](https://raw.githubusercontent.com/ankuanand/Blogs-Image/master/even%20eulerian.png)

Transverse
>A -> B -> C -> E -> B -> D -> C -> A 

We start and end up at the same node so the node should have a even degree
This is special kind of the Eulerian Path and this is known as **Eulerian Tour**

