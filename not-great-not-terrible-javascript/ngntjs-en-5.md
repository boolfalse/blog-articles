
## Node.js: thread pool, real life analogy (part 5/7)

### NGNTJS - part 5

---

In the previous [part 4](https://medium.com/@boolfalse/node-js-libuv-introducing-event-loop-523aa3444a63) of the NGNTJS article series, we looked at the libUV library, we had understanding about the event-loop mechanism.
This part will be about a feature called "thread pool" in [libUV](https://github.com/libuv/libuv).

<img src="https://i.imgur.com/Cuai8DB.png" style="width:100%;">

[Image from [unsplash.com](https://unsplash.com/photos/4ftI4lCcByM)]

Thread pool is designed to perform javascript blocking operations. As you may already know, there is one thread for JavaScript execution (as a runtime in Node.js as well), and only one operation is performed in that thread at a given moment in time. When blocking operations are encountered during code execution, such as the following methods:

- [fs](https://nodejs.org/api/fs.html) (fs.createReadStream(), fs.chown(), fs.copyFile())
- [crypto](https://nodejs.org/api/crypto.html) (crypto.scrypt(), crypto.sign(), crypto.verify())
- [dns](https://nodejs.org/api/dns.html) (dns.lookup(), dns.resolveCname() )

or many other asynchronous methods, additional threads are "opened" in order not to overload the work of the single thread. In fact, pooling and using each thread is quite resource-consuming, and this happens in cases where there is "no other option" in order not to block the work of the master thread.

<img src="https://i.imgur.com/VGrWoh5.png" style="width:100%;">
 
[Manufacturing single lane]

To better understand thread pooling from the master thread, let's try to use a similar analogy from real life:

Consider a senior factory worker who monitors the movement of packaged goods passing through the line, and he is responsible for the weight labels on the packaged goods. In the course of work, there are cases when the packaged product is quite heavy and that senior employee is not able to weigh it. For these cases, he has assigned junior assistants who can pick up that heavy package, weigh it on a large scale, tell the senior employee the weight so he can label the package and place it in the goods section of the line. Regardless of the weight, a junior employee is responsible for weighing any heavy item. Junior employees are young assistants who are not idle, but work in different sub-sections of the factory and are ready to help with any questions at any time of inquiry.
The senior employee is responsible for labeling, he follows the line, in case of heavy goods he asks assistants to help, and weighs light goods by himself. The senior employee generally tries not to ask for additional help from the assistants, since the assistant employees are also doing some work in different parts of the factory, perhaps following the senior employee to reach him in an emergency if necessary. However, according to the position, the senior employee has 4 assistants attached to him, who will come to help. Of course, the senior addresses them as needed, and as little as possible.
Thus, according to the labor regulations, appropriate action will be applied to the following cases registered during work:

- The senior appeals to 1 junior for only 1 heavy load.
- The senior applies to the 2 juniors at the same time for 2 heavy loads in a row.
- The senior applies to the 3 juniors at the same time for 3 consecutive heavy loads.
- The senior applies to the 4 juniors at the same time for 4 consecutive heavy loads.
- The senior cannot call on another assistant for more than 4 consecutive loads. It means that in such cases, 4 assistants take their loads in order to weigh them, and the one who finishes weighing earlier and tells the weight to the senior, continues to take the next heavy load without waiting for the others. And at some point, when one of them is freed and sees that there is no heavy load left, he immediately leaves with his affairs without waiting for the other assistants.

There are cases when the workload increases and the senior employee needs to use the help of additional junior employees. In such cases, there is an opportunity for the director to change the regulations and allow the senior employee to apply to more than 4 employees at the same time (of course, he can also reduce it).

This fictional example from real life was intended to make a comparison between the following terms:

- **master thread** - senior employee
- **thread** - every employee (basically it's about assistants)
- **thread pool** - application to support staff
- **UV_THREADPOOL_SIZE** = 4 - the number of helper assistants by default (this also depends on the OS)
- **Node.js runtime** - factory director
- **sync / async non-blocking operations** - loads that can be handled by a senior employee
- **async & blocking CPU-intensive operations** - loads that a senior employee assigns assistant employees to perform and report the result

Increasing UV_THREADPOOL_SIZE will reach a point where it no longer has an effect. That's why you need to know about computer threads.

The number of central processing units (CPUs) and cores (cores) of modern computers can vary depending on the physical structure, but they usually have a single CPU, while some servers as well as HPC (high-performance computing) systems can have multiple processors.
The number of cores per CPU can also vary greatly from 2 / 4 / 8 / 10 / 12 / 16 / 18 / 36 and more, but modern processors usually have 2 (dual-core) or 4 (quad-core) cores.
Modern CPUs often use a technology called hyper-threading, which allows each CPU core to provide multiple threads simultaneously, usually up to 2 (in some cases 4).

<img src="https://i.imgur.com/ElvOPsN.png" style="width:100%;">
 
[[Concurrency and Multithreading](https://www.logicbig.com/quick-info/programming/multi-threading.html)]

Now having a little understanding of the CPU-core-thread interrelationship, we can continue exploring the thread pool mechanism.

Let's consider a classic case - the work of Node.js on a modern computer, where there is a need for thread pooling. In the critical case, when Node.js needs to perform many CPU-intensive operations, maximum threads are provided by the cores to perform these operations. It means that in a quad-core CPU we will have 8 threads, and the thread pool mechanism can theoretically use those 8 threads at the maximum (in this case, full CPU resources will be used).
It is recommended to have UV_THREADPOOL_SIZE less than the number of threads possible in the given CPU, but the ideal case is when for a CPU with 4 cores (with the possibility of maximum 8 threads) we have UV_THREADPOOL_SIZE exactly 4, i.e. exactly the number of cores, in order to avoid OS workload.

It should also be noted that thread pool is not performed for all asynchronous operations.
For example, https.request() is a Network I/O operation, not a CPU-intensive operation, and it does not use a thread pool.
Instead of thread pooling, libUV tries to handle the task directly through the OS kernel using the Network Card (however, this has its limitations).
In fact, async methods in Node.js are handled by libUV, but they are handled in two different ways: either by the native async mechanism (e.g. [https.request()](https://nodejs.org/api/https.html#httpsrequestoptions-callback)) or by the thread pool ([fs.createReadStream()](https://nodejs.org/api/fs.html#filehandlecreatereadstreamoptions)).
At least if possible, libUV tries to avoid using thread pool, saving resources in the environment, and in critical cases, it uses additional threads, sticking to its function, which is to maintain non-blocking (async) work.

As we mentioned, one thread (single-threaded) is intended for JavaScript execution. But while saying this, we are talking about the JavaScript engine working in the browser. Of course, this also applies to Node.js, but it's also worth knowing that there are multi-thread runtime environments as well, such as Microsoft's [Napa.js](https://github.com/microsoft/napajs), which was discontinued in 2018. We will talk about that later in [part 7](https://medium.com/@boolfalse/javascript-language-engines-runtimes-environments-6fdab4e7759e).

In this part of the NGNTJS article series, we have some idea about thread pooling.
Next, in [part 6](https://medium.com/@boolfalse/node-js-event-loop-with-practical-examples-a-big-picture-fa0481512237) we will explore the workflow of the event-loop mechanism in a practical example. Also, there will be an introduction to Reactor Pattern.

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [GitHub](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [https://boolfalse.com/](https://boolfalse.com/)
