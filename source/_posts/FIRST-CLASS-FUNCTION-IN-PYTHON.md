---
title: Understanding First Class Function in Python
date: 2016-04-24 09:43:22
tags: [Python, Function, First-Class Function, Python Basic]
category:
- Python Fundamental
---

## Introduction

Function in Python are First- Class Objects. This means that they can be passed around and used just like any other data types. In Shorthand it's also known as "First-class function"
>Some of the characteristics of First Class Object Objects ?

  * Created at runtime
  * Assigned to a variable or element in a data structure
  * Passed as an arguments to a function
  * Returned as the result of a function

**Integers, Strings and Dictionaries are other examples of first- class objects in Python.**

## Created at Runtime
``` python
>>> def factorial(n): 
...  """return n!"""
...  return 1 if n < 2 else n * factorial(n - 1)
... 
>>> factorial(5)
120
>>> factorial.__doc__
'return n!'
>>> type(factorial)
<class 'function'>
>>>
```

In The above example we created a function at the runtime, __doc__ is one of several attribute of the function objects, and factorial is an instance of function class

## Assigned to a variable or element in a data structure
``` python
>>> fact = factorial
>>> fact
<function factorial at 0x7fdfb7f7fbf8>
>>> fact(4)
24
>>>
```

We can assign it a variable fact and call it through that name

## Returned as the result of a function 

``` python
>>> map(factorial, range(5))
<map object at 0x7fdfb7e93390>
>>>
```

In the above example we passed it to the map and it also returns a function as result, Which produces a below result when passed to list

## Passed as an arguments to a function

``` python
>>> list(map(factorial, range(5)))
[1, 1, 2, 6, 24]
>>>
```

Another example is the sorted built-in function: an optional key arguments lets you provide a function to be applied to each item for sorting.

``` python
>>> name = ['Ankur', 'Anand', 'Sam', 'John', 'KK', 'I']
>>> sorted(name, key=len)
['I', 'KK', 'Sam', 'John', 'Ankur', 'Anand']
```
Custom Reverse Function used for Sorting.

``` python
>>> def reverse_sort(word):
...  ''' reverse the word '''
...  return word[::-1]
... 
>>> reverse_sort('Ankur')
'ruknA'
>>> sorted(name, key=reverse_sort)
['I', 'KK', 'Anand', 'Sam', 'John', 'Ankur']
>>>
```
