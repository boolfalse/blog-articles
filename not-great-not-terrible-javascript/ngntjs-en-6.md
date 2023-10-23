
## Node.js: event-loop with practical examples. A big picture (part 6/7)

### NGNTJS - part 6

---

In the previous [part 5](https://medium.com/@boolfalse/node-js-thread-pool-real-life-analogy-e4574ef86c9b) of NGNTJS article series we had some understanding about thread pool with a practical real life example.
In this part, we will look at different examples (step by step) and through these examples we will try to understand how event-loop works in practice.

<img src="https://i.imgur.com/Cr2NSPb.png" style="width:100%;">
 
[Image from [unsplash.com](https://unsplash.com/photos/UEX8WkrdQgE)]

Before preparing this series of articles, and especially this part, a lot of material was studied, in which the following logical sequence can be observed: first, what is an event-loop, and then how it works.
Of course, there is no transition here either (previous: [part 5](https://medium.com/@boolfalse/node-js-thread-pool-real-life-analogy-e4574ef86c9b) and [part 4](https://medium.com/@boolfalse/node-js-libuv-introducing-event-loop-523aa3444a63) has some familiarity with libUV and event-loop), but as an author, I think it's worth starting to explore event-loop with practical examples, to understand how it works, and then get a bigger picture of what it actually is.

Example 1․ microtask and timer queues

```javascript
setTimeout(() => console.log("setTimeout 1"), 0);
setTimeout(() => {
  console.log("setTimeout 2");
  process.nextTick(() => console.log("setTimeout 2 nextTick"))
}, 0);
setTimeout(() => console.log("setTimeout 3"), 0);

process.nextTick(() => console.log("nextTick 1"));
process.nextTick(() => {
  console.log("nextTick 2");
  process.nextTick(() => console.log("nextTick 2 nextTick"));
});
process.nextTick(() => console.log("nextTick 3"));

Promise.resolve().then(() => console.log("Promise 1"));
Promise.resolve().then(() => {
  console.log("Promise 2");
  process.nextTick(() => console.log("Promise 2 nextTick"));
});
Promise.resolve().then(() => console.log("Promise 3"));
```

The result will be:

```log
nextTick 1
nextTick 2
nextTick 3
nextTick 2 nextTick
Promise 1
Promise 2
Promise 3
Promise 2 nextTick
setTimeout 1
setTimeout 2
setTimeout 3
setTimeout 2 nextTick
```

Example 2․ I/O queue and timer queue anomaly

```javascript
const fs = require("fs");

setTimeout(() => console.log("setTimeout"), 0);
fs.readFile(__filename, () => console.log("readFile"));
// for (let i = 0; i < 1000000000; i++) {}
```

The result will be something like this:

```log
setTimeout
readFile
// anomaly
readFile
setTimeout
```

Example 3․ I/O queue and I/O polling

```javascript
const fs = require("fs");

fs.readFile(__filename, () => {
  console.log("readFile");
});
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("Promise"));
setTimeout(() => console.log("setTimeout"), 0);
setImmediate(() => console.log("setImmediate"));

for (let i = 0; i < 1000000000; i++) {}
```

The result will be:

```log
nextTick
Promise
setTimeout
setImmediate
readFile
```

Example 4․ check queue & timer queue anomaly

```javascript
setTimeout(() => console.log("setTimeout"));
setImmediate(() => console.log("setImmediate"));
```

The result will be something like this:

```log
setTimeout
setImmediate
// anomaly
setImmediate
setTimeout
```

Example 5․ Close queue in the end

```javascript
const fs = require("fs");

const readableStream = fs.createReadStream(__filename);
readableStream.on("close", () => {
  console.log("readableStream");
});
readableStream.close();

setImmediate(() => console.log("setImmediate"));
setTimeout(() => console.log("setTimeout"), 0);
Promise.resolve().then(() => console.log("Promise"));
process.nextTick(() => console.log("nextTick"));
```

The result will be:

```log
nextTick
Promise
setTimeout
setImmediate
readableStream
```

You can also find examples on [GitHub Gist](https://gist.github.com/boolfalse/5f2c7160a6478da492634facc0c416e9).

In these examples, the microtask (nextTick and Promise), timer, I/O (I/O polling), check and close queues are sequentially passed.

Now, since here we already understand what the event-loop mechanism is, and we know the stages of work, we can look at the Node.js architecture in a "big" picture.
Using the keyword "Node.js architecture" on the internet, you can find many visuals, which are made by different authors according to certain requirements, to represent this or that component or the nature of the work they want.
Below I will post the visual made by me, which presents the work in Node.js Runtime in a high-level. It more oriented towards a logical chain than individual program blocks.

<img src="https://i.imgur.com/sHsjTvh.png" style="width:100%;">

[Interaction between libUV's event-loop and other components in the Node.js runtime]

Let's briefly describe the meaning of the new terms mentioned in the visual, that not yet explained in the articles before:

- **event-demultiplexer** - responsible for monitoring various I/O sources and receiving events. If the I/O source requires a blocking operation, event-DEMUX will pass the given task to the C/C++ field, where the given task will be executed. Once the task is finished, the C/C++ side will notify the event-dispatcher, which in turn will send it to the event-loop to be added to a specific queue.
- **event-dispatcher** - is responsible for event management, sending and receiving events
- **kqueue** - BSD-based mechanism for event notifications (macOS, FreeBSD, NetBSD)
- **IOCP** (input/output completion ports) - BSD-based mechanism for the event notifications (macOS, FreeBSD, NetBSD)
- **event ports** - Solaris-specific mechanism for the event notifications (Oracle Solaris, OpenSolaris)
- **epoll** - A Linux-specific mechanism for the event notifications

In addition, libUV provides other useful features in apart from above described features:

- **Asynchronous I/O** - libUV provides a common interface for asynchronous I/O operations across different OSs.
- **Signal handling** - libUV provides an API: Signal handlers are registered to handle OS signals such as SIGINT or SIGTERM.
- **File system operations** - libUV provides an API to perform file system operations such as file writing or file reading.
- **Network I/O** - libUV provides an API to perform network I/O operations, such as creating and managing TCP/UDP sockets.
- **Child processes management** - libUV provides an API for creating and managing child_processes.
- ...

Coming to this point, we can understand the idea of Reactor Pattern.

<img src="https://i.imgur.com/WytzyaP.png" style="width:100%;">

[Image from the mini-series [Chernobyl](https://www.imdb.com/title/tt7366338/)]

In this part of the series of NGNTJS articles we studied the event-loop mechanism through practical examples, got an understanding about the Reactor Pattern.
Next, in [part 7](https://medium.com/@boolfalse/javascript-language-engines-runtimes-environments-6fdab4e7759e) we will discuss JavaScript language, engine, runtime and environment. Then we will get to know modern JavaScript runtime/environments, such as Deno and Bun, and much more.

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [GitHub](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [https://boolfalse.com/](https://boolfalse.com/)
