---
title: Don't use Mutable as default Parameter in Python functions
date: 2016-04-24 12:06:04
tags: [Python, Function, First-Class Function, Python Basic]
category:
- Python Fundamental
---

## Introduction

>Examples use `python3` for `python2` use `func_defaults`

We frequently use `[]` as default parameter.
``` python
>>> def foo(a=[]):
...   pass
... 
>>> foo.__defaults__
([],)
>>> 
```
But this is not a good practice as this lead to debugging [rabbit hole](https://en.wikipedia.org/wiki/Rabbit_hole)

## Understanding Why ?

*The Default parameters are initialized during function definition time* which occur at either module load time or during program execution. As we can see in the above example the `foo` function was defined and it's default parameter shows the value of list which is empty `([],)`.

``` python
def foo(items, seq=[]):
  for item in items:
    if item >= 0:
      seq.append(item)
  return seq
```
Till now the only item in the `__defaults__` is an empty list as can be seen

``` python
print(foo.__defaults__) # ([],)
```

After first Invocation
```python
print(foo([2])) # [2]

print(foo.__defaults__) # ([2],)
```

From the output we can see that now the `__defaults__` doesn't shows that the default for seq which should be an empty list and this leads to `error` on subsequent invocations

``` python
print(foo([3])) # [2, 3]

print(foo([-1])) # [2, 3]
```
Why this happens because the list, being a mutable type, is still the same instance defined during function creation, but it is now populated from the interaction of the previous invocation.


## General Solution

Shift the creation from definition time to runtime.

*A Common idiom is to have the parameter default to None then check for that value in the body of the function*

``` python
def foo(items,seq = None):
  seq = seq or []
  for item in items:
    if item >= 0:
      seq.append(item)
  return seq


print(foo([2])) #[2]
print(foo([3])) #[3]
print(foo([-1])) #[]
```

**Point to Remember :** Don't use mutable types as default parameters

