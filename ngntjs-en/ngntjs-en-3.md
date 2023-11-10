
## JavaScript: Execution Context, Data Storing (part 3/7)

### NGNTJS - part 3

---

In the previous [part 2](https://medium.com/@boolfalse/javascript-engine-meaning-and-structure-7d904697cd97) of NGNTJS article series introduced the meaning and structure of JavaScript engines.
In this part, we'll go over how JavaScript works. We will present the Execution context and data storing, variables and functions (first-class citizens). We'll cover terms like the call stack, variable types, hoisting, and more.

<img src="https://i.imgur.com/rAtZenX.png" style="width:100%;">
 
[Image from [unsplash.com](https://unsplash.com/photos/a-room-with-wooden-walls-and-black-curtains-iiFnnqiPhFw)]

JavaScript is a single-threaded language, considering a thread here as a single sequential flow of control.
JavaScript is also a synchronous language, but has asynchronous capabilities that come out-of-the-box.
The Thread, which is created during the execution of JavaScript code, uses the Call Stack (stack of commands working on the Last In First Out principle) and Memory Heap (memory area for arrays, objects, functions).

Running Javascript code creates an environment where code transformation (interpretation and the rest) and execution takes place. That environment is called the Global Execution Context. It contains the current working code and everything related to it. In addition to the Global EC, each JavaScript function called has its own EC: Function Execution Context.

After the creation of any EC (Global EC or Function EC), 2 stages comes sequentially.

A. **Memory creation phase**

- A global object is created. For example, in Chrome running on the V8 engine, the global object is "window", and in Node.js is "global".
- "this" object is created and the reference of the global object is given to it as a value. For example, by calling "this" on any page in Chrome, we will get the same "window" object.
- Stores all variables (as undefined) and function references in the global EC (the entire function is stored in the memory heap).

B. **Execution phase**

- The code is executed line by line.
- During line-by-line operation, a corresponding EC is created for any function.

Let's take the following code block as an example and examine the 2 phases of the Global Execution Context:

```javascript
var x = 2;
var y = 3;
function multiply(a, b) {
  var m = a * b;
  return m;
}
var z = multiply(x, y);
```

Let's explain this example.

As already mentioned, at first the Global EC will be created (it will be "placed" in the Call Stack). In the creation phase, a memory space will be freed for the variable _x_ (memory allocation) and it will receive _undefined_ as a value in the memory heap.
The memory space for the variable _y_ will be freed and will be set to _undefined_ in the memory heap.
Then it will free up memory space for the _multiply_ function and get the entire function as a value in the memory heap. That is, treated as a _key:value_ pair, the function name will be stored as a reference to the corresponding function on the memory heap.
The memory space for the variable _z_ will be freed and will be set to _undefined_ in the memory heap.

In the second phase, which is execution phase, starting from the first line, _x_ will get 2 as a value, _y_ will get 3 as a value. Upon reaching the function, no execution will be happened, and the _multiply_ function will be invoked in the last line. Therefore, Function EC will be created.

In the EC of the _multiply_ function, with a similar analogy, at first the memory creation phase will be happen: memory will be allocated for _a_, _b_, and then for _m_ variables, and they will be given an _undefined_ value. Then the execution phase will happen, where _a_ will get 2 as a value, _b_ will get 3 as a value, then _m_ will get 6 as a value (2*3). The function will return the value from the Function EC to the Global EC and that Function EC will be removed from the Call Stack. Returning to Global EC, _z_ will get the returned value of 6. Finally, the Global EC will also be removed from the Call Stack and thus the program will be terminated (not talking about the work of [garbage collector](https://v8.dev/blog/trash-talk)).

The above process can be described with the following picture:

<img src="https://i.imgur.com/3Lr0qgJ.gif" style="width:100%;">

[Example of JavaScript Execution Context in action]

Using the debugger in modern web browsers, we can follow the execution phase of a given code, stepping through the code's commands. For example in Chrome it can be seen as shown in the image below:

<img src="https://i.imgur.com/sxiYysL.png" style="width:100%;">

[Example of execution phase in Chrome]

As you can see in the indicated part in the picture, the Call Stack contains an (anonymous) element at the bottom, which is the Global EC. And the _multiply_ function opened a new Function EC, which was added to the Call Stack, and what is shown in the picture is the moment when the thread works on the execution context of the function.

In any case, the Call Stack has a limit, and when it exceeds that limit, we will get an _Uncaught RangeError_, as shown in the bottom picture.

<img src="https://i.imgur.com/PwFpapL.png" style="width:100%;">
 
[Uncaught RangeError: Maximum call stack exceeded]

It should also be noted that there may be cases when we have an error during code execution, when the Call Stack will automatically be emptied. The image below shows the moment in the Call Stack in the console where the error was encountered:

<img src="https://i.imgur.com/CgvZpKp.png" style="width:100%;">
 
[Uncaught Error - automatic clearing of the call stack]

Now, to explore the uniqueness of the Execution Context on another example, we can modify the code and consider the following case:

```javascript
console.log(x, y, multiply);
var x = 2;
var y = 3;
var z = multiply(x, y)
function multiply(a, b) {
  var m = a * b;
  return m;
}
```

As we can see, the _multiply_ function is called before it is declared. This may be strange, but actually, since we already know about the two phases of the Execution Context, then we can understand that there can be no problem in this case, because in the first phase, _x_, _y_, and _multiply_ are initialized, then the execution phase comes, where _x_ and _y_ have the _undefined_ value, and the _multiply_ function have already been initialized.
It means that _console.log_ will print the following output:

<img src="https://i.imgur.com/BZrkpMK.png" style="width:100%;">

[Image from Chrome's Inspect]

Now that we have an understanding of the Execution Context, we can talk about **Hoisting**.

As we noticed, in the code written in the example, the variables _x_ and _y_ are declared with _var_. Let's try to modify the code once more and use _let_ instead of _var_ when declaring variables _x_ and _y_.

```javascript
console.log(multiply);
console.log(x, y);
let x = 2;
let y = 3;
var z = multiply(x, y);
function multiply(a, b) {
  var m = a * b;
  return m;
}
```

In this case, the browser will return _Uncaught ReferenceError_ (although it won't be a problem to use _multiply_), indicating that the error was registered as we already know in the Execution Context called _<anonymous>_, starting from the 13th symbol of line 2:

<img src="https://i.imgur.com/JLWUA5k.png" style="width:100%;">

[Image from Chrome's Inspect]

In fact, we received an error in the 2nd phase: in the execution phase, the JavaScript interpreter tries to log what is written in the input, it tries to find a variable registered with the name _x_ in the first (initializing) phase - there is no such one, then it tries to find a reference to the function registered with the name _x_ - it does not exist either, so not finding any reference, it sends back a _ReferenceError_.

Without talking about Hoisting at this point, we can confirm that there is a difference between _var_ and _let_ , at least in terms of the fact described so far.

_let_ and _const_ are in JavaScript since ES6 (ECMAScript 2015). One of the differences between _let_ and _var_ is hoisting. _var_ variables are hoisted in the Global scope in the creation (or we can say initialization) phase and are available already in the execution phase, already assigned with the value _undefined_.
Although many sources state that _let_/_const_ variables are not hoisted, in fact they are hoisted, but to the _Script scope_, and therefore are not available during the execution phase. That's why, unlike _var_, _let_ and _const_ are block-scoped variables.
Below is an example when we have a debugger enabled in the execution phase, which on the given line (stopped and has not yet tried to execute that line) already records that the variables defined by _let_ are hoisted in the _Script scope_. Let's note once again that their access is closed in the execution phase, that's why we get the following error:
_caught ReferenceError: Cannot access 'x' before initialization at index.html:13:13_

<img src="https://i.imgur.com/ndQwAKa.png" style="width:100%;">

[Image from Chrome's Inspect]

So we got some idea about JavaScript hoisting by knowing about execution phases beforehand.
Actually _let_/_const_ and _var_ variables have many other differences as well, which can be read about in the following [freeCodeCamp article](https://www.freecodecamp.org/news/var-let-and-const-whats-the-difference/) and [the following stackoverflow thread](https://stackoverflow.com/a/11444416/7574023).

Now let's talk about how JavaScript manages data and where it is stored.

Depending on the given value, primitive or reference, it is stored in stack (not to be confused with Call Stack) or heap, respectively.

In relatively low-level languages such as C and C++, the developer himself chooses what memory can be allocated for the data. In relatively higher-level languages, such as JavaScript and Python, it automatically allocates memory when an object is created, and then automatically cleans up that memory when it is no longer needed. This is done using the built-in _garbage collector_ tool. Languages mentioned at the beginning are so-called _non-garbage-collected languages_, and in that case memory must be managed manually, otherwise it can lead to a memory leak in some cases. In any case, even in modern JavaScript, you should avoid saving unwanted references, which can lead to memory leaks.

It is known that the creator of JavaScript, Brendan Eich, did not have a garbage collector written in the demo version after the end of his famous 10-day sprint, which was added later.

As you probably know, modern JavaScript has 7 primitive data types and 3 reference types:

**Primitive Types**: String, Boolean, Number, undefined, Symbol, BigInt
it is stored in the stack and can be accessed by itself (without a link)

**Reference Types**: Array, Function, Object
values are stored in the heap and their links are stored in the stack, through which access can be obtained

Now, already knowing that in the case of a specific example function, the function itself is stored in the heap, and the reference to the function (with which you can have access to that function) is stored in the stack, we can better understand the previously written example of Hoisting. There was described an example of how it comes that the function is registered in the initialization/creation phase of the execution context and is already available in the execution phase, even though the _var_ and _let_/_const_ variables behaved differently.

Now let's consider an example where we have an object and we perform certain manipulations on it:

```javascript
// variables case
let a = 2;
let b = a;
a = 5;
console.log(b); // ---------------------- 2
```

```javascript
// objects case
let car = {
  maxSpeed: 200,
  wheelsInch: 17
};
let anotherCar = car;
car.maxSpeed = 240;
console.log(anotherCar.maxSpeed); // ---- 240
anotherCar.wheelsInch = 18;
console.log(car.wheelsInch); // --------- 18
```

As we can see, in the case of a variable, the data itself changes its value, and the object (referenced data) is controlled by the reference to it. For example, when we create a new object by overriding to it the previously existing object, then in this case a deep copy is not made (the content of the old object), but only a new link is created, which has the same access to the given object as had the previously declared one. It means that both _car_ and _anotherCar_ are just accesses to an object on the heap.
For the described example, we will have the following image:

<img src="https://i.imgur.com/IPHaiWJ.png" style="width:100%;">

[Pictured is a specific moment where no attributions have been made yet]

Finally, perhaps as an additional material, we have a lecture [link](https://www.youtube.com/watch?v=Aa_OWn03mDo&list=PLEzQf147-uEpvTa1bHDNlxUL2klHUMHJu&index=116&t=256s) dedicated to work in JavaScript, which may be a circumstance that will allow "to see" JavaScript functions differently.

<img src="https://i.imgur.com/JmjCmvW.png" style="width:100%;">
 
[[Douglas Crockford](https://www.crockford.com/): Fun With Functions]

Below are links to useful resources that did not appear directly in this section, but are related to this topic:

- [JavaScript Execution Context](https://luigicruz.dev/blog/execution-context) [Apr, 2021]
- [JavaScript Execution Context - How JS Works Behind The Scenes](https://www.freecodecamp.org/news/execution-context-how-javascript-works-behind-the-scenes/) [Feb, 2022]

This part of the NGNTJS article series introduced the Execution Context in JavaScript, its memory creation and execution phases, described the operation of the call stack through practical examples, as well as variable and reference types.
Next, in [part 4](https://medium.com/@boolfalse/node-js-libuv-introducing-event-loop-523aa3444a63) we will consider the libUV library, as well as be familiar with the operation of the event-loop mechanism.

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [GitHub](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [https://boolfalse.com/](https://boolfalse.com/)
