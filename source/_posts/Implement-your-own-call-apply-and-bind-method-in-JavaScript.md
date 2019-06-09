---
title: "Implement your own  call(), apply() and bind() method in JavaScript"
date: 2018-06-15 10:07:05
thumbnail: https://cdn-images-1.medium.com/max/1600/1*KJbe8d4I_rKjkaxh0mBppA.png
tags: [JavaScript]
category:
  - JavaScript
---

It completely makes sense to not reinvent the wheel at your work, but it’s also of great significance sometime to build some smaller wheels yourself for the purpose of learning. Simulating these small wheels in a comprehensive way does enhance your own learning of the same.

> “What I Cannot create, I do not understand” — Richard Feynman

So let’s try to simulate the method `call`, `apply` and `bind` in JavaScript.

## Call Method

First of all, let’s take a look at what original `call` method is.

![MDN definition of call](https://cdn-images-1.medium.com/max/1600/1*KJbe8d4I_rKjkaxh0mBppA.png)

Example of `call`:

```js
function showProfileMessage(message) {
  console.log(message, this.name);
}
const obj = {
  name: "Ankur Anand"
};
showProfileMessage.call(obj, "welcome ");
```

The above example will give us output `welcome Ankur Anand` .

So if we analyze the basic principle of a call function from above, we notice these things.

1.  Calling the prototype function call changes the pointing of this. i.e Function call in the above became `obj.showProfileMessage`.

2.  Whatever arguments we have passed to `showProfileMessage.call` should be passed to original `showProfileMessage` as `arg1, arg2,...` .

3.  Does not cause side effects to `obj` and `showProfileMessage` Function, i.e calling `call` doesn’t modify the original `obj` or `showProfileMessage` by any mean.

Let’s try to implement the first step of the idea of call. As the `call` method prototype method of `Function` Object. Our custom `myOwnCall` will be also be attached to the `Function` prototype.

```js
Function.prototype.myOwnCall = function(someOtherThis) {
  someOtherThis.fnName = this;
  someOtherThis.fnName();
};
```

With that, we have achieved the first step of this, If you are wondering how take a look closely at this.

`showProfileMessage.myOwnCall(obj, "welcome ");`

What happens when we call something like above, `showProfileMessage` is an object (Function is also an object in JavaScript) on which we are calling the method `myOwnCall` inherited from prototype, so inside our `myOwnCall` method currently the `this` variable will be pointing to the object that is `showProfileMessage` and this is what our function reference is, so we added a function referencing to `showProfileMessage` to new passed `obj` or `this` value from the `myOwnCall` with `someOtherThis.fnName = this;` and then simply called the function in the next line.

Let’s now implement the second idea of `call`, to call the function with the passed parameters of variable length.

Before that let’s look at the `eval` function in JavaScript.

> The `**eval()**` function evaluates JavaScript code represented as a string.

`eval` takes a string representing a javascript expression, statement, or sequence of statements. The expression can include variables and properties of existing objects.

```js
Function.prototype.myOwnCall = function(someOtherThis) {
  someOtherThis.fnName = this;
  var args = [];

  // arguments are saved in strings, using args
  for (var i = 1, len = arguments.length; i < len; i++) {
    args.push("arguments[" + i + "]");
  }

  // strings are reparsed into statements in the eval method
  // Here args automatically calls the Array.toString() method.

  eval("someOtherThis.fnName(" + args + ")");
};
```

Let’s get the third idea of the `call` method. “To not cause any side effect” and that exactly what we have violated till now in our implementation. `someOtherThis.fnName = this;`

We are adding `fnName` property to `someOtherThis` assuming that `someOtherThis` does not have a property named `fnName` in advance, we should avoid this. We can use a symbol of `es6` but we will avoid it as of now. We will use `Math.random` to create a unique property `id` for the object, and should delete this property after execution.

Also if we look at the MDN definition we see that the `call` function should return a value with the result of specified `this` and `arguments`. Also if the value of `someOtherThis` is `null` or `undefined` it should be replaced with a global object (`window` for browser and `global` for node.js).

Let’s check our final implementation of the same.

```js
Function.prototype.myOwnCall = function(someOtherThis) {
  someOtherThis = someOtherThis || global;
  var uniqueID = "00" + Math.random();
  while (someOtherThis.hasOwnProperty(uniqueID)) {
    uniqueID = "00" + Math.random();
  }
  someOtherThis[uniqueID] = this;
  const args = [];
  // arguments are saved in strings, using args
  for (var i = 1, len = arguments.length; i < len; i++) {
    args.push("arguments[" + i + "]");
  }

  // strings are reparsed into statements in the eval method
  // Here args automatically calls the Array.toString() method.
  var result = eval("someOtherThis[uniqueID](" + args + ")");
  delete someOtherThis[uniqueID];
  return result;
};
```

Please note that the above function is still not completely safe from a side effect if, for example, the called function iterates over the keys of `‘this’` it would get a different result with the above implementation than the native implementation. (Thanks to [LukaLight](https://www.reddit.com/user/LukaLightBringer) for this point).

## Apply Method

Let’s look at the `apply` method definition as on MDN.

![apply method mdn](https://cdn-images-1.medium.com/max/1600/1*rHxYRBmAhxudu-LQmGqJ5Q.png)

It’s very much same as `call` method with only difference being its takes an `array` like object as arguments. So without any further ado, let’s look at its complete implementation.

```js
Function.prototype.myOwnApply = function(someOtherThis, arr) {
  someOtherThis = someOtherThis || global;
  var uniqueID = "00" + Math.random();
  while (someOtherThis.hasOwnProperty(uniqueID)) {
    uniqueID = "00" + Math.random();
  }
  someOtherThis[uniqueID] = this;

  var args = [];
  var result = null;
  if (!arr) {
    result = someOtherThis[uniqueID]();
  } else {
    for (let i = 1, len = arr.length; i < len; i++) {
      args.push("arr[" + i + "]");
    }
    result = eval("someOtherThis[uniqueID](" + args + ")");
  }

  delete someOtherThis[uniqueID];
  return result;
};
```

## Bind Method

Again we will start by looking at the MDN definition of the same.

![bind method as on MDN
](https://cdn-images-1.medium.com/max/1600/1*DfTEFPWxtRZNxIELLs4fYg.png)
The general characteristic of the bind function is as follow from mdn.

1.  The bind method creates and returns a `new function`, called a **bound function**. This **bound function** wraps the original function object.


```js
Function.prototype.myOwnBind = function(newThis) {
  if (typeof this !== "function") {
    throw new Error(this + "cannot be bound as it's not callable");
  }
  var boundTargetFunction = this;
  return function boundFunction() {
    return boundTargetFunction.apply(newThis);
  };
};
```


2. Look at this line, from mdn definition “**arguments to be prepended to the arguments provided to the bound function, when invoking the target function**”.

I will try my best to break it down step by step for you.

```js
var person = {
  lastName: "Anand"
};

function fullName(salutaion, firstName) {
  console.log(salutaion, firstName, this.lastName);
}

var bindFullName = fullName.bind(person, "Mr");

bindFullName("Ankur");
```

If we run the above code the output we get is `Mr Ankur Anand`

So If you look carefully the argument “_Mr_” was provided at the time of creating a **bound function** `bindFullName` while argument “_Ankur”_ was provided at the time of invoking the `bindFullName` which in turn invokes the target function. So the argument “_Mr_” was prepended to the arguments list when we called the `bindFullName` with an argument “_Ankur”._ This is what the definition at the MDN for `args1, args2, ...` means.

Let’s us try to implement the same in our own `myOwnBind` method.

```js
Function.prototype.myOwnBind = function(newThis) {
  if (typeof this !== "function") {
    throw new Error(this + "cannot be bound as it's not callable");
  }
  var boundTargetFunction = this;
  var boundArguments = Array.prototype.slice.call(arguments, 1);
  return function boundFunction() {
    // here the arguments refer to the second time when we call the target function returned from bind
    var targetArguments = Array.prototype.slice.call(arguments);
    return boundTargetFunction.apply(
      newThis,
      boundArguments.concat(targetArguments)
    );
  };
};
```

We are using `Array.prototype.slice.call` because `arguments` [are not array but an array-like object.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments)

Are we done with bind? No, We have still missed one piece from the mdn definition of the bind for `thisArg`.

> The value is ignored if the bound function is constructed using the [new](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new "The new operator creates an instance of a user-defined object type or of one of the built-in object types that has a constructor function.") operator.

What it means to say is we need to ignore the passed in `this` value while creating the **bound function** when the bound function is called with a `new` operator. Taking the example given above `new bindFullName("Ankur")` will give us output as`Mr Ankur undefined`.

MDN has a nice polyfill of bind that takes care of the new operator scenario. What it basically does is set a transit constructor fNOP, so that the bound function and `bind()`the function call is on the same prototype chain, because calling the bound function with the new operator involves the passing of the prototype chain.

You can [check the Polyfill of the bind at mdn.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#Polyfill)

![MDN Pollyfill of bind](https://cdn-images-1.medium.com/max/1600/1*moB8J7pRUSd4YCgRg6uWVA.png)

Hope this article proves helpful to you at providing some insight of call, apply and bind in JavaScript as it’s has been for me 😅. If you find any issue or typo please let me know. 😃
