---
title: Call By Sharing
date: 2016-05-10 18:52:39
tags: [JavaScript, Python, Java]
category:
- Programming Paradigms
---

## Introduction

We all are familiar with the `pass by value or call by value` and `pass by reference or call by reference` and how does one programming language evaluates this depends upon the  [Evaluation Strategy](https://en.wikipedia.org/wiki/Evaluation_strategy) specified by that programming language. But in the mist of these two terms we have missed an important aspect of this strategy what is known as `call by sharing`.

## Understanding `Call By Sharing`

Let's see what is.

### Call by Value
We all have seen code like this in `C`.
``` c
#include <stdio.h>

void fuc(int a, int b){
  printf("%u\n", &a );
  printf("%u\n", &b);
}

int main(){
  int a = 10;
  int b = 30;
  printf("%u\n", &a);
  printf("%u\n", &b);
  fuc(a,b);
  return 0;
}
```


The Output on my machine:

```
3441025176
3441025180
3441025148
3441025144

```


As you can see that in `call by value` the value gets copied(new address for new location of value). **But this approach becomes a bit problematic in term of performance issue when function arguments is not primitive value, but a complex structure like `struct`**

Example:
For People who are not familiar with `C` Think `struct` as a class of `OOP` without any method in it.

``` c
typedef struct point3D
{
 int x;
 int y;
 int z;
}point_3D;

void fuc(Struct a){
  printf("%u\n", &a );
}

int main(){
  point_3D point = {0, 0, 0};
  printf("%u\n", &point); // get the address of the very first element
  fuc(point);
  return 0;
}
```

The Output is as follow
```
1197172472
1197172440
```

As you can see each individual members gets a new memory address, which can lead to serious performance issue in term of memory while using large number of structure.

### Call by Reference

In turn the `pass by reference` receives not a value copy, but the address directly related with object from the outside. Any Change of parameter inside the function **even assignment of new value** is affected on object outside. As the exact address of this object is related with formal parameter.

``` c
#include <stdio.h>


void swap(int* a, int* b){
int temp = *a;
*a = *b;
*b = temp;
}


int main(){
  int a = 10;
  int b = 20;
  swap(&a, &b);
  printf("%d %d\n",a, b );
  return 0;
}
```

Output:
```
20 10
```
We can see that the a and b outside the swap function gets affected.

### Call By Sharing

We often hear a common phrase that
>Objects are passed by reference; primitives are passed by value.

Lets have a look a following Python code (Even though the Example has been illustrated in Python for Simplicity the same hold true for some other language like `Java` and `JavaScript` code sample below):

``` python
>>> x = []
>>> x
[]
>>> def mutateX(alist):
...   alist.append("Python List is Mutable")
... 
>>> mutateX(x)
>>> x
['Python List is Mutable']
>>> 
```

Since everything in Python is an `Object` We can argue that the above behavior is as expected as we will get from the `pass by reference`.

Ok How about this behavior in case of `assignment`:
``` python
>>> def reref(alist):
...   alist={'a':65}
... 
>>> reref(x)
>>> x
['Python List is Mutable']
```

Even Though we modified the `mutable` list object into a `dictionary` but the `x` is still a `list` object. (If it would have been call by reference the x should now point to a dictionary) Why so ?

Lets Dig a little bit.

As Python Implements a [Reference Counting Mechanism](http://localhost:4000/2016/04/23/Object-Mechanism-in-Python-part1/#Reference-Counting-Mechanism) which count the number of times an Object has been referred.

For Example
``` Python
>>> a=0x1234 # large value as small value gets pooled
>>> sys.getrefcount(a)
2
>>> b=a
>>> sys.getrefcount(a)
3
>>> c=b
>>> sys.getrefcount(a)
4
>>> c
4660
>>> a
4660
>>> b
4660
>>> 
```

When we write `a=0x1234` an `Integer Object` is created and `a` gets reference to it(precisely bind it to name 'a'). But do you see that for each other assignment the reference count of the `Integer object` gets bumped up.

**Understanding Why we get ref count more than 1 initially.**
Here is the what  [Python Doc](https://docs.python.org/2/library/sys.html#sys.getrefcount)
>The count returned is generally one higher than you might expect, because it includes the (temporary) reference as an argument to  getrefcount().

means that when we call getrefcount(), the `reference is copied by value into the function's
arguments`, temporarily bumping up the object's reference count.

Remember the Term **Reference is copied by value into the function's arguments** because this plays an important role in `call by sharing`

Assigning b to another object decreases the reference count.Also since Integer is immutable in python we can't change what initial value of b was. we can just rebind it to another Object.

``` python
>>> b=0x12345 
>>> sys.getrefcount(a)
3
```

That's What happens when you pass an argument to a function the **Reference is Copied by value**. 

As you can see below
``` Python
>>> x = []
>>> x
[]
>>> def mutateX(alist):
...   alist.append("Python List is Mutable")
... 
>>> mutateX(x)
>>> x
['Python List is Mutable']
>>> import sys
>>> sys.getrefcount(x)
3
```

The `x` reference count becomes `3` one higher than what expected because the `alist` gets a copied reference,like `alist=x=[]`.
But what happens in the case of `assignment` we detached the from `alist=[]` to `alist={'a':65}`. Due to this the x i.e `list []` never got modified.


Sample JavaScript Code

``` JavaScript
> function mutate(obj){
... obj.name = 'Mutated';
... }
undefined
> var my_color = {
... name : 'Blue'};
undefined
> my_color
{ name: 'Blue' }
> mutate(my_color)
undefined
> my_color
{ name: 'Mutated' }
> 
```

Since We passed the mutable object the JavaScript will mutate it in the above.
But in the case of assignment and immutable object the changes won't reflect to the caller as in below case.

``` JavaScript
> function chanageColor(col){
... col = 'red';
... }
undefined
> var colr = 'green';
undefined
> chanageColor(colr)
undefined
> colr
'green'
> 
```

## The Curious Case of `Java`

Java is also `Call by Sharing`. May be `pass by value where value is the reference copy` describes the Evaluation Strategy better in `Java` than simply saying `Java` is always pass by value.

