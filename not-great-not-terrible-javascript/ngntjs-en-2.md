
## JavaScript Engine: meaning and structure (part 2/7)

### NGNTJS - part 2

---

In previous [part 1](https://medium.com/@boolfalse/browsers-javascript-creation-74f5cb4f8ff8) of NGNTJS article series we talked about the history of browsers, the origins of JavaScript, the emergence of the ECMA standard.
In this part we will present the meaning and structure of JavaScript engines.

<img src="https://i.imgur.com/20M5uGG.png" style="width:100%;">

["Check Engine"]

All web browsers have a JavaScript Engine. It is a software component that optimizes and executes JavaScript code. At the beginning of the history of web browsers, this component was mainly just an interpreter, but modern engines are much more than an interpreter, they have additional components that perform optimization, and also use JIT compilation, which improves performance.
The function of JavaScript engine is to run JavaScript code regardless of whether it is run in a browser, Node.js, CLI environment or any IoT device.
We will consider JavaScript engines in more detail in [part 7](https://medium.com/@boolfalse/javascript-language-engines-runtimes-environments-6fdab4e7759e), and in this part we will try to get sufficient general information.

Currently, different browsers work with different engines. Some of the famous ones are:

- [**SpiderMonkey**](https://spidermonkey.dev/) - the first engine ever created by the same Brandon Eich that was first used by Netscape Navigator, then became [open source](https://searchfox.org/mozilla-central/source/js/src). Now it's running Mozilla Firefox.
- [**V8**](https://v8.dev/) - It is an [open source](https://github.com/v8/v8) engine released by Google ([issues page](https://bugs.chromium.org/p/v8/issues/list)). This can be used standalone, or it can be embedded in other C++ applications, such as the Node.js runtime (which runs JavaScript code on the server), that also take advantage of this.
- [**ChakraCore**](https://github.com/chakra-core/ChakraCore) - Microsoft's own [open source](https://github.com/chakra-core/ChakraCore) engine used by the former IE and current Edge (before ECMA standards, the language was called JScript). One of its special features is that it compiles scripts on a separate CPU core, in parallel with the web browser.
- [**JavaScriptCore** (JSC)](https://github.com/WebKit/WebKit/tree/main/Source/JavaScriptCore) - It is an [open source](https://github.com/WebKit/WebKit/tree/main/Source/JavaScriptCore) (own [repo](https://opensource.apple.com/source/JavaScriptCore/)) engine created by [Apple](https://developer.apple.com/documentation/javascriptcore). This uses Safari as well as all other Apple [WebKit engines](https://webkit.org/project/) (rendering engines). As with V8, this way JavaScriptCore can be used separately in Swift, Objective-C, and C-specific applications. It is also used as an engine in Bun (we will talk about this later).

Currently, there are other open source JavaScript engines that can be used in different environments.

- [Boa](https://github.com/boa-dev/boa) - Boa is an embeddable and experimental Javascript engine written in Rust. Currently, it has support for some of the languages.
- [Hermes](https://github.com/facebook/hermes/) - A JavaScript engine optimized for running React Native.
- [Yantra](https://github.com/yantrajs/yantra) - JavaScript Engine for .NET Standard.

Although different browsers approach their JavaScript engines slightly differently, they all essentially do the same thing. Even though the solutions may have some differences, their differences create a natural competition that has led to significant changes and development over time.
Anyway, let's describe the general picture of the operations of the modern JavaScript Engine:

<img src="https://i.imgur.com/l8jD0SY.png" style="width:100%;">

[Image built with [draw.io](https://app.diagrams.net/)]

The image above is only a very general view of the JavaScript engine, it describes how JavaScript code works. In addition to the fact that JavaScript engines differ from each other, their differences also change over time, as it undergoes certain changes during development. But just to understand, you can consider the scheme shown in the picture above.

**Source code**

In step 1 we have JavaScript/TypeScript code.
This can be code for different environments, such as a web browser, a server, a CLI, or any IoT device.

**Parser**

Parser is designed for transforming code into an abstract syntax tree. Parser checks the syntax of the code, and if there is an error, it returns an error and thus stops further work.
In case of valid syntax, Parser builds the Abstract Syntax Tree.
It is worth to listen to the following [talk](https://www.youtube.com/watch?v=Fg7niTmNNLg) (the corresponding [presentation](https://docs.google.com/presentation/d/1b-ALt6W01nIxutFVFmXMOyd_6ou_6qqP6S0Prmb1iDs/)) at JSConf EU 2017 about Parser work in V8. Here the speaker talks about the need for parsers, explains the work of the parser on an example, talks about caching, lazy and eager modes of parsing, about the work of parsing modes in the function context, about the problems that arise during the parsing stage, etc.

**Abstract Syntax Tree (AST)**

AST is also used to build the syntax tree of many other programming languages and tools (like CSS, HTML, GraphQL, Java, JSON, Markdown, PHP, Scala, SQL, Regular Expressions, etc.). A visual example of an implementation of this can be seen here: [AST explorer](https://github.com/fkling/astexplorer).

**Interpreters/Compilers**

> _Hint. if necessary, you can get acquainted with the interpreter, compiler, JIT-compiler below._

After obtaining the abstract syntax tree, it is passed to the 4th step, where the interpreter must process the tree and obtain an intermediate bytecode representation from it.
Bytecode is the output state of the JavaScript engine when the code is represented as an instruction set. This is the stage when the programmer has no direct influence. At this stage, the variables and functions of the script code are stored and used in the registers (main memory) and in the accumulator (current memory), and they are used to fulfill the requirements of the instruction set.

**Intermediate Representation, a.k.a. Bytecode**

The bytecode received by the interpreter is actually the form that needs to be transformed into machine code (in block 7) and then passed directly to the CPU.
But, in fact, we see that the interpreter does not directly transfer it from the 4th block to the 7th block. That being said, the abstract tree is not immediately transformed into machine code. The reason is that before the machine code can be generated, there needs to be some intermediate representation (IR, also known as bytecode) on which some work needs to be done, such as JIT compiling (with block 6).
Bytecode is engine-specific, it means different engines have their own bytecode syntax and instruction sets.

One way to see the bytecode of the written JS code is to use the node cli's "--print-bytecode" argument. Let's look at an example to get a clear idea. We have an index.js file with the following content:

```javascript
function sum() {
  var first = 10;
  var second = 5;
  var third = first + second;
  return third;
}

console.log(sum());
```

To see the piece of bytecode associated with the sum function (the entire bytecode is quite large), we can run the following:

```shell
node --print-bytecode --print-bytecode-filter=sum index.js
```

as a result we will get the following:

```yaml
[generated bytecode for function: sum (0x095c31356c51 <SharedFunctionInfo sum>)]
Bytecode length: 13
Parameter count 1
Register count 3
Frame size 24
OSR urgency: 0
Bytecode age: 0
   31 S> 0x95c31357690 @    0 : 0d 0a             LdaSmi [10]
         0x95c31357692 @    2 : c4                Star0 
   50 S> 0x95c31357693 @    3 : 0d 05             LdaSmi [5]
         0x95c31357695 @    5 : c3                Star1 
   67 S> 0x95c31357696 @    6 : 0b f9             Ldar r1
   73 E> 0x95c31357698 @    8 : 39 fa 00          Add r0, [0]
         0x95c3135769b @   11 : c2                Star2 
   98 S> 0x95c3135769c @   12 : a9                Return 
Constant pool (size = 0)
Handler Table (size = 0)
Source Position Table (size = 12)
0x095c313576a1 <ByteArray[12]>
15
```

The following can be understood from the extracted data (bold font is the meaning of the letters):

- LdaSmi [10]
  **l**oa**d** **sm**all **i**nteger **10** into the **a**ccumulator
- Star0
  **st**ore **a**ccumulator value to the **r0** register
- LdaSmi [5]
  **l**oa**d** **sm**all **i**nteger **5** into the **a**ccumulator
- Star1
  **st**ore **a**ccumulator value to the **r1** register
- Ldar r1
  **l**oad **r1** **r**egister value into the **a**ccumulator
- Add r0, [0]
  **add** whatever it is in **r0** **r**egister into the **0** index of the **a**ccumulator
- Star2
  **st**ore **a**ccumulator value to the **r2** register
- Return
  **return** the current value of the accumulator

In the table below, let's show the sequence of instructions and the values of the accumulator and registers at each step:

```markdown
| Instruction | Accumulator | Register r0 | Register r1 | Register r2 |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| <START>     |             |             |             |             |
| LdaSmi [10] |     10      |             |             |             |
| Star0       |     10      |     10      |             |             |
| LdaSmi [5]  |      5      |     10      |             |             |
| Star1       |      5      |     10      |      5      |             |
| Ldar r1     |      5      |     10      |      5      |             |
| Add r0, [0] |     15      |     10      |      5      |             |
| Star2       |     15      |     10      |      5      |      15     |
| Return      |     15      |     10      |      5      |      15     |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| <END>       |     15      |             |             |             |
```

Regarding bytecode, there is an interesting article written by a member of the V8 team: [Understanding V8's Bytecode](https://www.fhinkel.rocks/posts/Understanding-V8-s-Bytecode), and [here](https://www.alibabacloud.com/blog/javascript-bytecode-v8-ignition-instructions_599188) the bytecode of V8's Ignition interpreter is considered in detail on various examples.

**JIT (just-in-time) compiler**

Although step 6 is optional, most JavaScript engines currently have a JIT compiler component. Different engines apply their own optimization compilers, which may be more than one, to generate the bytecode and optimize performance.

<img src="https://i.imgur.com/dtfApv4.png" style="width:100%;">

[Images taken from the links below]

You can learn more about them from the employees of their own teams:

- **V8**: [Fanziska Hinkelmann - Introduction to JavaScript engines and performance](https://www.youtube.com/watch?v=WBkMm19ziUI)
- **JSC**: [Michael Saboff - JavaScriptCore, many compilers make this engine perform](https://www.youtube.com/watch?v=mtVBAcy7AKA)

Briefly, a JIT compiler is a tool that generates machine code directly from bytecode, performing optimization. Currently, for example V8 engine uses [TurboFan](https://v8.dev/docs/turbofan) as a JIT compiler, SpiderMonkey engine uses [WarpMonkey](https://hacks.mozilla.org/2020/11/warp-improved-js-performance-in-firefox-83/) (formerly IonMonkey) as a JIT compiler.

**Machine Code**

It is necessary to take into account the fact that the machine code, being [assembly language](https://en.wikipedia.org/wiki/Assembly_language), is specific to the architecture of the machine (in the case of ARM, Intel-based or other processors), it means the machine code will be different for machines with different architectures, that is why it is necessary to have that so-called intermediate bytecode.

<img src="https://i.imgur.com/ZFV6zzf.jpg" style="width:100%;">

[Image from [unsplash.com](https://unsplash.com/photos/QndYCQc_a3g)]

**Compiler, Interpreter, JIT-compiler; real life analogies**

- **_Compiler_**
  This is software that translates source code written in a high-level language (for example, C++) into machine code (specific to the machine architecture) "once and at all".
  To translate, it performs a static analysis of the entire source code, checks for errors, and generates an executable or binary file that can be executed by the machine independently of the compiler. That file can already be considered as a separate object, which will not need to be compiled in order to be executed later.
- **_Interpreter_**
  This is mostly a program already implemented with an interpreted language, or in the case of JavaScript, its engine. In the classic case, when each line contains one instruction, it reads each command line by line, translates it into machine language and executes the machine code corresponding to that command (with a slight difference in the case of JavaScript).
  In the case of JavaScript, the difference is that the interpreter turns the source code line by line into an intermediate language (IR - intermediate representation) instead of immediately doing it. Once the source code has been fully interpreted into an intermediate language (that's bytecode), it is then fully translated into machine language and executed.
  Modern interpreters analyze the source code not in its entirety, but on the principle of receiving the next command, and can perform optimization at the time of receiving the next command.
  In the Java language, source code is also translated into bytecode (for example, using the javac compiler), then the bytecode is translated into machine code by the interpreter and the JIT compiler for the JVM in the given environment to be executed.
- **_JIT (just-in-time) compiler_**
  JIT compiler can be considered as a hybrid of compiler and interpreter. This is software that dynamically analyzes the source code and compiles block by block into machine language before executing, making optimizations between blocks. When the source code is completely compiled and there is complete machine code, execution comes.

<img src="https://i.imgur.com/u5Pts3Y.png" style="width:100%;">
 
[Image from [americangirl.com](https://www.americangirl.com/products/american-girl-baking-cookbook-dtd87)]

- **_Compiler - real life analogy_**
  Imagine you have a cake recipe written in a language you don't understand. You give the recipe to a translator who reads the entire recipe, understands the instructions, and translates them into a language you understand. The translated recipe is then given to you and you can bake the cake by following the instructions. So you can bake the cake with all the translated instructions.
- **_Interpreter - real life analogy_**
  Imagine you have a recipe for baking a cake written in a language you don't understand, and your friend knows that language. This time you call to your friend for help. Your friend reads the instructions line by line and translates them to you sequentially. After listening to the translation of each instruction, you follow the current instruction, and the resulting cake is baked as a result of the steps taken sequentially.
  Not necessarily, but in modern life it is possible that your friend also has some cooking knowledge. In this case, for example, when the N-th instruction is _"add 3 spoons of sugar"_ and the N+1 instruction is _"add another 2 spoons of sugar"_, your friend considers these two instructions as one instruction and formulates the optimized instruction as _"add 5 spoons of sugar"_ for you, thereby saving the number of operations and reducing the preparation time.
- **_JIT compiler - real life analogy_**
  Imagine you have a recipe that is written in an unknown language, but you have a personal assistant who can simultaneously translate that language and also has excellent culinary knowledge. Your assistant reads it block by block, translates it and writes it down, performs some optimizations on it (more active than your friend in the case of interpreters), and tells you the optimized version of the next few instructions. At each subsequent instruction, while you follow the instruction, your assistant continues to make notes in his notebook, perform calculations, think, and prepare to tell you the next block of instructions.
  For example, your assistant might say _"wash 50 strawberries, remove the tails, cut them in half, and place those 100 strawberry halves cut side up on the cake layer"_, whereas in the non-optimized case you would wash 1 strawberry, remove the tail, cut it in half, and place the 2 strawberry halves cut side up on the cake layer, and this sequence of instructions would do 50 times because you don't know the future instructions. However, if you know that in the next 49 steps you have to wash strawberries, you wash those 50 strawberries in a container at once.
  In a theoretical case, it is possible to imagine that a recipe with 20 instructions can be translated for you by your assistant and presented as an example in the form of 5 instructions.
  Thus, your assistant helps you to optimize calculations, save resources and reduce time.

Below are links to useful resources that did not appear directly in this section, but are related to this topic:

- [An Introduction to Speculative Optimization in V8](https://ponyfoo.com/articles/an-introduction-to-speculative-optimization-in-v8) [Nov, 2017]
- [Attacking JS engines: Fundamentals for understanding memory corruption crashes](https://www.sidechannel.blog/en/attacking-js-engines) [Apr, 2023]
- [How JavaScript Works: Under the Hood of the V8 Engine](https://www.freecodecamp.org/news/javascript-under-the-hood-v8/) [Aug, 2020]
- [The Mysterious Realm of JavaScriptCore](https://www.cyberark.com/resources/threat-research-blog/the-mysterious-realm-of-javascriptcore) [Mar, 2021]
- [Talks by fhinkel on YouTube](https://www.youtube.com/playlist?list=PL65pp6Tpk690HkOh324FqtFYyxFVnEFEX)
- [How JavaScript works: Optimizing the V8 compiler for efficiency](https://blog.logrocket.com/how-javascript-works-optimizing-the-v8-compiler-for-efficiency/) [Sep, 2019]
- [Just In Time (JIT) Compilers - Computerphile](https://www.youtube.com/watch?v=d7KHAVaX_Rs) [Nov, 2022]
- [Ode to the V8](https://medium.com/@lucygibbmar/ode-to-the-v8-ff2a4c394ac8) [Apr, 2019]

> Appendix: for those who are interested in low-level topics (compilers, interpreters, etc.), useful resources available on YouTube such as [MIT 6.172 Performance Engineering of Software Systems, Fall 2018](https://www.youtube.com/playlist?list=PLUl4u3cNGP63VIBQVWguXxZZi0566y7Wf) and [Compilers and Interpreters 2022](https://www.youtube.com/playlist?list=PLlUGRprNEgH-FVK5NekKPg__7ySjeI3gn) lectures, and [DragonBook 2nd edition](https://www-2.dc.uba.ar/staff/becher/dragon.pdf) for a professional approach.

This part of the NGNTJS article series introduced the meaning and structure of JavaScript engines.
Next, in [part 3](https://medium.com/@boolfalse/javascript-execution-context-data-storing-172dcdf721f9) we will move on to studying the work of JavaScript. We will present the Execution context and data storing, variables and functions (first-class citizens). We'll cover terms like the call stack, variable types, hoisting, and more.

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [GitHub](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [https://boolfalse.com/](https://boolfalse.com/)
