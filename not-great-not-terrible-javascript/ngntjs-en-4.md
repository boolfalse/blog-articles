
## Node.js: libUV, introducing event-loop (part 4/7)

### NGNTJS - part 4

---

In the previous [part 3](https://medium.com/@boolfalse/javascript-execution-context-data-storing-172dcdf721f9) of NGNTJS article series we talked about Execution Context, creation (initialization) and execution phases, Hoisting, primitives and references (7 primitives + 3 referenced data-types) and data storing.
In this part, we will consider the libUV library, as well as get acquainted with the operation of event-loop.

<img src="https://i.imgur.com/2YlhS1x.png" style="width:100%;">

[Image taken from the following [NodeCongress23 talk](https://youtu.be/aR_zxqoeBpQ?t=339)]

Now we know that JavaScript is a single-threaded language, where a thread "opens" an execution context (Global EC and, if necessary, Function ECs as well), which hoists its primitives and referenced data on the stack and heap respectively.
In addition, we also know that JavaScript is a synchronous language, it means the execution is performed by passing each command by blocking them.

However, as we know, apart from JavaScript engine, there are other components as wall that ensure the implementation of asynchronous operations.
There are some "bridges" (Web APIs) between the JavaScript and the async support part so that it is possible to pass data from one to the other to perform certain async operations.
Often the commands used to perform these asynchronous operations are confusing, and sometimes some people think they are part of JavaScript, but they are Web APIs (for example implemented for the browser) to perform these asynchronous, non-blocking operations and send back responses.

There are quite a few [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API) that use the event-loop mechanism to handle asynchronous operations in the browser. The list of Web APIs is constantly expanding and changing over the time as new technologies are developed and adopted, however below are some of the most common ones:

- setTimeout
- setInterval
- fetch
- requestAnimationFrame
- addEventListener
- geolocation API
- WebSockets
- Web Workers
- IndexedDB
- FileReader API
- WebRTC
- Web Audio API
- Service Worker API
- Notification API
- MediaDevices.getUserMedia()
- Intersection Observer API
- Resize Observer API
- Mutation Observer API
- …

<img src="https://i.imgur.com/5O0ETjD.png" style="width:100%;">
 
[[https://github.com/libuv/libuv#readme](https://github.com/libuv/libuv#readme)]

[**libUV**](https://github.com/libuv/libuv) (UV = Unicorn Velociraptor) is an open source and cross-platform library written in C, mainly for asynchronous I/O operations. It was originally written for Node.js (used in other environments as well) to support async operations and provide non-blocking execution. thread-pool, event-loop and other mechanisms are used here for doing that.
event-loop works both in the browser environment and in the Node.js runtime, and their work has a certain commonality for both of them.
However, one of the main differences between event-loop mechanism in Node.js and browsers (we can consider the Chrome browser as an example, as the V8 engine is implemented there and in Node.js as well) is that in browsers the event-loop works in the same thread where the browser UI works, while in Node.js the event-loop works in a separate thread.
In general, although there are differences between the event-loop mechanisms of Node.js and browsers, they both serve the same purpose of allowing non-blocking, asynchronous operations.

There is an interesting [conference-talk](https://www.youtube.com/watch?v=8aGhZQkoFbQ) on YouTube, where the speaker talks about the event-loop in browsers, where you can see and understand the browser part of event-loop quite clearly through examples.
[The video conference was back in 2014, but I think it's worth posting as a pretty successful talk]
In order not to get a repetition here, or a mere translation, I think we can consider the information provided on that video as a part of event-loop topic (in browsers).
But just to save time, here are some important key-parts where event-loop is graphically presented in the browser:

<img src="https://i.imgur.com/bvTCpKn.gif" style="width:100%;">

[example using call stack without event-loop (sync operations)]

<img src="https://i.imgur.com/pGfYNBD.gif" style="width:100%;">

[example of using event-loop using setTimeout() timer-type Web API]

The speaker has provided other examples that can be implemented using [latentflip/loupe](https://github.com/latentflip/loupe). Another link to the similar tool at [CodeSandbox](https://codesandbox.io/s/6lgw1).

While writing this series of articles, I have studied the materials produced by various authors on the topic, where some visuals about event-loop are found, such as:

<img src="https://i.imgur.com/oSB2AP3.jpg" style="width:100%;">

[Event Loop and the Big Picture - NodeJS Event Loop Part 1](https://blog.insiderattack.net/event-loop-and-the-big-picture-nodejs-event-loop-part-1-1cb67a182810) [Apr, 2017]

<img src="https://i.imgur.com/4cRiGqN.jpg" style="width:100%;">

[The JavaScript Event Loop: Explained](https://towardsdev.com/event-loop-in-javascript-672c07618dc9) [Apr, 2021]

<img src="https://i.imgur.com/tSpcWxW.jpg" style="width:100%;">

[Event Loop in NodeJS - Visualized](https://medium.com/@mmoshikoo/event-loop-in-nodejs-visualized-235867255e81) [Nov, 2021]

<img src="https://i.imgur.com/LvqWWhm.jpg" style="width:100%;">

[What you should know to really understand the Node.js Event Loop](https://medium.com/the-node-js-collection/what-you-should-know-to-really-understand-the-node-js-event-loop-and-its-metrics-c4907b19da4c) [Jul, 2017]

<img src="https://i.imgur.com/7sxaDki.png" style="width:100%;">

[Synchronous vs Asynchronous JavaScript - Call Stack, Promises, and More](https://www.freecodecamp.org/news/synchronous-vs-asynchronous-in-javascript/) [Sep, 2021]

<img src="https://i.imgur.com/iziFRGx.png" style="width:100%;">

[NodeConf EU | A deep dive into libuv - Saul Ibarra Coretge](https://youtu.be/sGTRmPiXD4Y?t=640) [Nov, 2016]

<img src="https://i.imgur.com/fHp38vG.png" style="width:100%;">

[Node.js Tutorial - 42 - Event Loop](https://www.youtube.com/watch?v=L18RHG2DwwA&list=PLC3y8-rFHvwh8shCMHFA5kWxD9PaPwxaY&index=43) [Jan, 2023]

Pictures with such visuals can be found in many places. Although at first glance these are different from each other, in fact their logic is the same: to show the workflow of the event-loop. As a more suitable image (according to subjective circumstances) in the next part we will use an image similar to the last visual.

event-loop is one of the features of libUV library written in C, but not the only one. In general, this is a mechanism whose task is to manage JavaScript asynchronous operations. In addition to libUV, in the modern tech world there are few other software which implementing the idea of event-loop. For example, [Deno](https://deno.land/) created by the same Node.js creator [Ryan Dahl](https://tinyclouds.org/), uses another asynchronous event-driven platform called [Tokio](https://tokio.rs/) written in Rust, instead of libUV written in C.

Deno, as well as other modern alternatives, will be discussed in more detail in [part 7](https://medium.com/@boolfalse/javascript-language-engines-runtimes-environments-6fdab4e7759e), and in this part we will limit ourselves to this. Considering that we have some introductory material about the even-loop in the browser environment, in the following parts we will study the event-loop mechanism already in the Node.js runtime (V8 engine + libUV).

In this part of the series of NGNTJS articles, we looked at the libUV library, got acquainted with the operation of the event-loop mechanism.
Next, in [part 5](https://medium.com/@boolfalse/node-js-thread-pool-real-life-analogy-e4574ef86c9b) we will have some idea about thread pool, and for that we will have a practical example from real life.

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [GitHub](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [https://boolfalse.com/](https://boolfalse.com/)
