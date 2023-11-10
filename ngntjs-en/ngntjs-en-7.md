
## JavaScript: language, engines, runtimes, environments (part 7/7)

### NGNTJS - part 7

---

In the previous [part 6](https://medium.com/@boolfalse/node-js-event-loop-with-practical-examples-a-big-picture-fa0481512237) of NGNTJS article series we studied the event-loop mechanism using some practical examples and had understanding about the Reactor Pattern.
In this part, having already been informed about some terms, we will discuss about JavaScript language, engine, runtime, environment. Then we will get to know modern JavaScript runtimes/environments, such as Deno and Bun, and not only.

<img src="https://i.imgur.com/98GtbyS.png" style="width:100%;">

[Image from JSConfEU23 [following talk](https://youtu.be/aR_zxqoeBpQ?t=890)]

So far we have talked about the JavaScript language, in particular the standard ECMAScript, which until the first decade of 2000s was only considered a browser-based scripting language.

Having already some information from previous parts of this article series, we know that JavaScript (TypeScript) language running in the browser is converting to the machine code to run the commands that are intended for the lower levels, that we already discussed in detail in [part 2](https://medium.com/@boolfalse/javascript-engine-meaning-and-structure-7d904697cd97).
So the JavaScript engine's main functionality is to interpret the JavaScript code using the call stack (the storage that used for storing sequence of commands) and the memory heap (the storage for primitive and reference types that are needed during the code execution).

As we already know, different companies usually have their own versions of JavaScript engines. As an example, we can mention the [V8](https://chromium.googlesource.com/v8/v8) released by Google in 2008, which works in Chrome browser. Other browsers also have their own JavaScript engines but in all modern browsers the JavaScript language itself conforms to the ECMAScript standard with certain variations from different companies.

<img src="https://i.imgur.com/HqrOUO9.png" style="width:100%;">
 
[The official mirror of the V8 Git repository on [GitHub](https://github.com/v8/v8#readme)]

We also know that modern browsers often use various Web APIs, such as the well-known setTimeout(), setInterval(), relatively recently added fetch(), WebSockets, WebRTC, etc.
It means that these properties do not belong to the JavaScript language itself. And that are actually outside of the JavaScript engine's scope.
In addition to the methods described above, those methods need to use an event-loop and some necessary components associated with it.
And all of this, which is outside of the JavaScript language (engine) itself, includes the JavaScript runtime.
It means that the main components of JavaScript runtime are:

- **JavaScript engine**: the call stack, the memory heap, and the language itself that manages them.
- **WebAPI**s: setTimeout(), setInterval(), setImmediate(), fetch(), etc.
- **Event-loop** and related mechanisms: toolkit for running WebAPIs and many other asynchronous operations.

In the early 2000s, when JavaScript still only ran in browsers, providing a dynamic interface for websites, engineers tried to get a standalone version of JavaScript runtime that would work outside of the browser environment. It can be assumed that the V8 engine released by Google in 2008 is the basis for [Node.js](https://github.com/nodejs/node#readme) being released by [Ryan Dahl](https://tinyclouds.org/) in 2009.

<img src="https://i.imgur.com/TAYYCV9.png" style="width:100%;">
 
[Node.js [official website](https://nodejs.org/en)]

So V8 is a JavaScript engine written in C++, based on all the necessary tools have been collected to run JavaScript outside the browser environment. And Node.js is a technology that allows to run JavaScript outside the browser environment, using it both in the server-side and as a CLI tool.
For this reason, Node.js is considered a runtime environment. In this sense, V8 can be considered as a framework for Node.js.

In [part 2](https://medium.com/@boolfalse/javascript-engine-meaning-and-structure-7d904697cd97) we have mentioned some popular JavaScript engines, now it would be appropriate to mention following:

- At a time, there were also attempts to have Node.js analogs using other engines: [node-chakracore](https://github.com/nodejs/node-chakracore) (Node.js on ChakraCore) and [spidernode](https://github.com/mozilla/spidernode) (Node.js on top of SpiderMonkey), but these are no longer maintained.
- Usually, if we use Node.js with its V8 engine on the server, then a lightweight engine can be used in IoT devices intended for smaller loads, for example [jerryscript](https://github.com/jerryscript-project/jerryscript) (ultra-lightweight JavaScript engine for the Internet of Things) or [Duktape](https://github.com/svaarala/duktape) (embeddable Javascript engine with a focus on portability and compact footprint).

As an "alternative" to Node.js, there are currently other JavaScript runtime environments, such as [Deno](https://deno.land/).

<img src="https://i.imgur.com/kwq1eV8.png" style="width:100%;">

[Image from deno [official website](https://deno.land/)]

Deno is the second popular project created by Ryan Dahl, which was officially announced at the 2018 [JSConfEU talk](https://www.youtube.com/watch?v=M3BM9TB-8yA) by him. At [the following conference](https://www.youtube.com/watch?v=1gIiZfSbEAE) in 2019, the author highlights the following key features:

- Built on V8 engine (like Node.js).
- It executes TypeScript and JavaScript, has an embedded TypeScript compiler. Node.js implements a separated [compiler](https://www.typescriptlang.org/docs/handbook/compiler-options.html) for TypeScript, using the [typescript](https://github.com/microsoft/TypeScript#readme) npm package. [V8 Snapshots](https://v8.dev/blog/custom-startup-snapshots) are used to make the TS compiler run faster.
- Initially it was written in Go, then rewritten and now written in [Rust](https://www.rust-lang.org/) (Node.js is written in C++). Compiling performs the following transformation:
  JavaScript -> TypeScript -> Rust
- [Tokio](https://github.com/tokio-rs/tokio) is used for the event-loop (as an analogue to [libUV](https://libuv.org/) in Node.js).
- It is cross-platform (like Node.js).
- Works with single executable design (not like this in Node.js).
- By default it has a secure execution mechanism for importing external resources, such as requesting URLs, opening sockets, etc (not available in Node.js). Unlike Node.js, where we could install 3rd party resources that could do unwanted actions (for example, read the contents of confidential files and send them out), here by design any action that needs permission is necessarily forced to get the appropriate permissions from the admin (in this case, the developer).
- With the CLI argument, it is necessary to specify the resources that require permission to access, for example, with "--allow-read=/tmp" we allow reading the "/tmp" directory, or with "--allow-net=google.com" we allow connecting to google.com (not available in Node.js). Instead of "npm start" in Node.js, we have the following (as a sample):
  deno run https://bit.ly/deno-bronto --allow-net --allow-read
- Compared to Node.js, not all Browser APIs are supported, Deno's purpose is different.
- By design, Deno's 3rd party module system is distributive, as they are imported from relative or absolute URLs (pre-installed in Node.js), for example:
  import { serve } from "https://deno.land/std/http/server.ts";
  this allows to avoid complex algorithms of "node_modules" and also not to have a package.json.
- Packaging system is similar to Node.js, it fetches necessary modules at first, then uses the cached ones at other times, until we try to update them.
- If npm was considered as an add-on in Node.js, in Deno npm is implemented as npm-client.
- Unlike Node.js, where the file can also be used as a module (multiple submodules in it) and import specific submodules from the exported module, in Deno it is strict, the file can only be used as a module:
  Module == File == URL
- Unlike monolithic Node.js, Deno is a collection of Rust crates that allows for standalone executables. [Cargo-TypeScript integration crate](https://crates.io/crates/deno_typescript) is presented as a sample, which allows you to compile TS in V8 Snapshots and have a quick startup.
- Deno team has also released Deno [Fresh framework](https://github.com/denoland/fresh). Shortly, it is little more than [express.js](https://expressjs.com/) for Node.js. It has built-in front-end rendering support that uses [preact](https://preactjs.com/) and [JSX](https://react.dev/learn/writing-markup-with-jsx).

<img src="https://i.imgur.com/DQiXqDw.png" style="width:100%;">
 
[Image from [deno.com](https://deno.com/deploy)]

In addition, I think it's worth having some updates here from the 2023 NodeCongress [JavaScript Conferences by GitNation](https://www.youtube.com/watch?v=LVEGRj3RZSA):

- Deno 2.0 has a KV (key-value) built-in database based on SQLite. This makes it possible to have a ready-made database with already implemented tools without using additional databases.
  - KV stores JavaScript objects;
  - necessary get, set, list, delete commands are available;
- Until [Deno 2.0](https://github.com/denoland/deno/milestones) is released, you can use [the following tool](https://dash.deno.com/playground/getting-started-with-kv) to work with KV.
- Deno team has prepared the [Deno-Deploy tool](https://deno.com/deploy/pricing) to encourage the distribution idea, which allows to deploy and have a real working app.
- Deno-Deploy runs on top of FoundationDB and has the following features:
  - ability to integrate public and private GitHub repos;
  - works in 35+ edge locations worldwide;
  - free *.deno.dev subdomain and custom domains are provided;
  - free and automatic HTTPS / TLS;
  - unlimited production deployments and previews;
  - up to 10ms/request CPU provisioning;

Deno's philosophy is also to encourage the distributive idea. According to the creator, the idea is the same:

- **Node**: Restricting programs to **async I/O primitives** help developers build **optimal local servers**.
- **Deno**: Restricting programs to **distributed cloud primitives** help developers build **optimal global servers**.

It should be noted that according to Ryan Dahl, Deno 2.0 was supposed to be released in the summer of 2023, but it is already delayed, but even so, the last described points refer specifically to Deno 2.0.

As an "alternative" to the Node.js and Deno projects, other JavaScript runtime environments such as [Bun](https://github.com/oven-sh/bun) have recently become popular.

<img src="https://i.imgur.com/xJwpRl2.png" style="width:100%;">

[Image from Bun [official website](https://bun.sh/)]

Bun is an open-source project created by [Jarred Sumner](https://twitter.com/jarredsumner). During the 2022 [meeting](https://www.youtube.com/watch?v=eF48Ar-JjT8) the author highlights following key features:

[Note: because the software changes over time, that's why I won't mention here exactly how many times or by what percentage Bun is faster than this or that function of Node.js. The author often tweets on his page about that.]

- Bun is a modern all-in-one JavaScript runtime environment. Bun has the basic toolkit to start JS/TS code faster than Node.js. It has a built-in transpiler for JS/TS code, a built-in npm package manager, a front-end dev server, and of course a runtime.
- Implemented more optimized [Node.js-API](https://nodejs.org/api/addons.html#node-api) such as fs.copyFile and fs.writeFile methods, TextEncoder.encode, buffer.toString('bas64') etc., which work faster.
- _bun install_ installs npm packages faster.
- _bun run_ starts package.json scripts faster.
- [Node.js SQLite](https://github.com/WiseLibs/better-sqlite3) is faster in Bun: [bun:sqlite](https://bun.sh/docs/api/sqlite).
- Bun uses [JavaScriptCore](https://trac.webkit.org/browser/trunk/Source/JavaScriptCore) engine, unlike Node.js which uses [V8 engine](https://chromium.googlesource.com/v8/v8.git).
- Bun is mostly written in [Zig](https://ziglang.org/) unlike Node.js, which is written in C++.
- The author considers the previous two points as the basis of Bun's fast work, but in addition to them, he also mentions the following reasons:
  - Bun uses system calls much more flexibly;
  - Bun uses modern technologies such as [SIMD](https://arxiv.org/abs/2109.10433) (Single Instruction Multiple Data), which allows maximum use of CPU speed capabilities (now [applied](https://simdutf.github.io/simdutf/) also in Node.js);
  - quite a lot of time was spent on testing and optimization;
- [Bun 1.0](https://twitter.com/jarredsumner/status/1678424677629464576) is scheduled to be released this year (September 2023), which will be a stable version with as few bugs as possible;

Overall, Bun, being written in Rust, brings with it a new philosophy, as the author notes:
[_"in Bun your JavaScript code isn't I/O bound, it's system-call bound."_](https://youtu.be/eF48Ar-JjT8?t=264)

I think it's also worth mentioning about JavaScript runtime created by Microsoft [Napa.js](https://github.com/Microsoft/napajs/wiki/introduction).

<img src="https://i.imgur.com/rX89wnz.png" style="width:100%;">
 
[[microsoft/napajs](https://github.com/microsoft/napajs)]

Although it has not been maintained since 2018 and work on it has stopped, a significant feature of this runtime, unlike Node.js and alternative runtimes, it was designed to handle multi-threaded, CPU-intensive tasks. There is an [opinion](https://news.ycombinator.com/item?id=33354665), that Napa is mostly abandoned and predates [Node.js v10](https://nodejs.org/en/blog/release/v10.5.0) having native multi-threaded support and worker_threads. Anyway, it's worth knowing that in Napa.js the multi-thread concept is implemented by running V8 engines isolated from each other, each occupying the corresponding thread. It was possible to use it both as a Node.js module and as a separate standalone service, not being a Node dependency.
There is an article about napa.js [here](https://medium.com/codeinsights/starting-parallel-programming-in-node-js-with-napa-js-ef80e20ec6c2).

Below are links to useful resources that did not appear directly in this section, but are related to this topic:

- [Node vs Deno vs Bun: The search for a better JavaScript runtime environment](https://patagonian.com/blog/node-vs-deno-vs-bun-javascript-runtime/) [Oct, 2022]
- [Choosing a JavaScript Runtime for 2023: Node vs. Bun vs. Deno](https://softwaremill.com/choosing-a-javascript-runtime-for-2023-node-vs-bun-vs-deno/) [Mar, 2023]
- [Bun vs. Node.js](https://refine.dev/blog/bun-js-vs-node/#prerequisites) [Jun, 2023]
- [What is Deno?: Deno vs Node.js](https://www.knowledgehut.com/blog/web-development/what-is-deno-difference-between-deno-nodejs) [Jul, 2023]

In this part of the NGNTJS article series, we have discussed about JavaScript language, engine, runtime, and environment. We got information about Node.js alternatives, such as Deno and Bun JavaScript runtime/environments, and more.

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [GitHub](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [https://boolfalse.com/](https://boolfalse.com/)
