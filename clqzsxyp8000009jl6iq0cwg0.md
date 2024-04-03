---
title: "Multi-Threading and Concurrency in NodeJS"
seoTitle: "Multi-threading and concurrency in NodeJs"
seoDescription: "A thread is an execution context that consists of a set of instructions for the CPU to evaluate. Multi-threaded apps are faster by context-switching"
datePublished: Thu Jan 04 2024 22:51:56 GMT+0000 (Coordinated Universal Time)
cuid: clqzsxyp8000009jl6iq0cwg0
slug: multi-threading-and-concurrency-in-nodejs
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1704408312439/59514760-d998-40c2-bdaf-1abedff447aa.png
tags: nodejs, multithreading, concurrency, event-loop, parallelism

---

# Introduction

The importance of acquiring fundamental knowledge cannot be overemphasized. This statement holds in almost all facets of life if not all. In this article, we will explore some software engineering fundamentals and how they could be applied to a project. You will learn about:

* Multi-Threading
    
* Asynchronous programming
    
* NodeJs Event loop
    
* Concurrency and Parallelism
    

The reader is expected to have knowledge about NodeJs and its [asynchronous programming](https://de-marauder.hashnode.dev/handling-asynchronous-operations-in-javascript) paradigm.

# Case Study

I encountered a peculiar bug in a project I’ve been working on. 

## What was the issue? 

I had a long-running task (required around 1-2 mins execution time) that needed to be run when an ExpressJS API endpoint was called. The expected behavior was that when the endpoint is called, some database operations are done to obtain certain data then a response is sent back to the client to close the connection. The long-running job is started using the data retrieved from the database. A simplified version of the buggy code is given below

```javascript
// index.js
import express from "express"

const app = express();

const pr = new Promise((resolve) => {
  return resolve()
})

const performDBOperation = () => {
  const largeNumber = 1_000_000_000_000_000_000_000_000;
  return largeNumber
}

const longRunningJob = (largeNumber) => {
  for (let i = 0; i <= largeNumber; i++) {
    if (i === largeNumber) {
      console.log(i)
    }
  }
}

app.get('/async', async (req, res) => {
  console.log("hit");
  const data = await pr.then(() => performDBOperation());
  res.json({})
  pr.then(() => {
    longRunningJob(data)
    console.log("Job done");
  })
})

app.listen(4000, () => {
  console.log("============== App started =================")
})
```

In the above code, I instantiate an express application and define functions to mock a long-running job and a database operation. I then define the endpoint for handling the job. The controller is async so it is wrapped by a promise and treated asynchronously on execution. The database call is done and a response is sent back to the client before the long-running job is invoked asynchronously by wrapping it in a promise.

We can run this server and call the endpoint using the commands below.

```bash
# You can run both commands in different terminal sessions for a better experience.
# & allows the command to run in the background. TO cancel it run 'fg 1' and then cancel normally
node index.js &
curl localhost:4000/async
```

When the endpoint is called the first time, it returns the response pretty quickly, closing the connection and starting the job. That’s all good. However, when a second request is made, the request enters a pending state. Try it out. It only gets executed when the asynchronous job is done executing. This behavior repeats itself with all subsequent requests.

## So, What’s going on?

Debugging this issue would go ahead to lay a bunch of misconceptions I had about NodeJs bare. Before now I had erroneously thought that NodeJs processed successive requests to a server in parallel. Meaning that all requests were being executed simultaneously. I had thought that was what it meant for processes to be handled concurrently. Also, I had thought that asynchronous processes would always wait for synchronous processes. But boy was I wrong!

To properly explain what went wrong, let’s look at some definitions

### Parallelism

The concept of parallelism as regards computing is quite simple. It means that multiple tasks are being executed or processed simultaneously. 

> **Parallel computing** is a type of [computation](https://en.wikipedia.org/wiki/Computing) in which many calculations or [processes](https://en.wikipedia.org/wiki/Process_(computing)) are carried out simultaneously.[\[1\]](https://en.wikipedia.org/wiki/Parallel_computing#cite_note-1) Large problems can often be divided into smaller ones, which can then be solved at the same time.
> 
> Source: [https://en.wikipedia.org/wiki/Parallel\_computing](https://en.wikipedia.org/wiki/Computing)

### Concurrency

> In [computer science](https://en.wikipedia.org/wiki/Computer_science), **concurrency** is the ability of different parts or units of a [program](https://en.wikipedia.org/wiki/Computer_program), [algorithm](https://en.wikipedia.org/wiki/Algorithm), or [problem](https://en.wikipedia.org/wiki/Problem_solving) to be [executed](https://en.wikipedia.org/wiki/Execution_(computing)) out-of-order or in [partial order](https://en.wikipedia.org/wiki/Partial_Order), without affecting the outcome.
> 
> Source: [https://en.wikipedia.org/wiki/Concurrency\_(computer\_science)](https://en.wikipedia.org/wiki/Computing)

Concurrency is closely related to parallelism but in this case, the tasks are not exactly being executed at the same time. They just appear to be. The computer will process concurrent tasks by context switching which means that it performs one operation for some time pauses it and performs another operation for some time pauses it and continues this routine until all tasks are executed. Parallel computing on the other hand does not require context switching as the computer will perform all tasks simultaneously. No task is paused for the other.

By naive evaluation, one can assert that this means that parallel computing is faster and they would be right. It is important to note that parallel computing is only possible because computers now have multiple cores which means they have multiple CPUs and as such all CPUs could potentially be engaged to run tasks in tandem. This is opposed to the former architecture where only single core systems (1 CPU) existed and since the CPU could only really evaluate one instruction at a time, context switching was necessary to boost performance resulting in concurrent processing.

### What is a Process?

A process is essentially a running program. A program is a set of instructions written to a file that can be parsed and understood by the CPU. When a program is running, a thread is started (the main thread) and the execution of the process is tied to this thread. If required, the program could spin up other threads to handle side jobs. It is important to note here that the threads are scoped to processes and dependent on them. A process has a defined resource (memory and CPU) space and can only make use of these resources.

### What is a thread?

A thread is a sequence of instructions in a process. It is an execution context. Remember concurrency? Threads offer a way to implement it. Essentially threads allow a process to break off tasks into execution contexts that the CPU can switch into. It is important to note that the process provides a shared memory and CPU space for threads and as such information can be sent between threads easily. Processes on the other hand do not share memory spaces. They are isolated and independent of each other in that regard.

For a more in-depth understanding of processes and threads, you should check out this [discussion](https://stackoverflow.com/questions/5201852/what-is-a-thread-really) on StackOverflow and this [video](https://www.youtube.com/watch?v=Tcdp3RVcnOI).

### NodeJS Event Loop

The event Loop in NodeJS is an interesting topic that I won’t be able to do justice to in this article. However, I will touch on the relevant bits. The nodeJs documentation provides an excellent read on this topic [here](https://nodejs.org/en/guides/event-loop-timers-and-nexttick).

> The event loop is what allows Node.js to perform non-blocking I/O operations — despite the fact that JavaScript is single-threaded — by offloading operations to the system kernel whenever possible.
> 
> Source: [https://nodejs.org/en/guides/event-loop-timers-and-nexttick](https://en.wikipedia.org/wiki/Computing) 

The Event Loop is the backbone of nodeJS. It provides the event-driven avenue with which nodeJS handles program execution. When you run a javascript file with node, it spins up a single-threaded process by default, instantiates an event loop, and begins to stack instructions into it scheduling them for execution. Different phases in the event loop handle different types of operations from scheduled instructions (timers) to callbacks and regular instructions. The event loop iterates and jumps through its different phases executing instructions (it does this by sending them to the call stack a memory space for keeping track of function calls) in a predefined order.

The event loop is the reason why asynchronous programming in JavaScript works the way it does. By default, it executes tasks/instructions sequentially (the file is read as a FIFO queue and the instructions are executed using the call stack). However, it can also defer the execution of certain tasks using `callbacks` and `promises` as well as schedule execution of other tasks using functions like `setTimeout` and `setInterval`. These types of tasks ([asynchronous tasks](https://de-marauder.hashnode.dev/handling-asynchronous-operations-in-javascript)) are skipped and sent to the back of the line (or scheduled) while normal tasks (synchronous) tasks continue execution when NodeJS executes a javascript program. Basically, synchronous tasks get added to the call stack first and then the asynchronous tasks are added to the stack once it's empty. This is an oversimplification of what happens but hopefully, you get the picture. Please refer to the [docs](https://nodejs.org/en/guides/event-loop-timers-and-nexttick) for a more in-depth description.

Alright, we’ve talked about a bunch of stuff, but how does that help us solve our problem?

### Problem Discussion

As has been stated above, Javascript is single-threaded. NodeJs provides a way to run it in a non-blocking manner because otherwise all instructions would have to be executed sequentially and you’ll have to wait (no asynchronous execution) for blocking IO operations like Disk reads and network calls. 

Back to our problem.

When Node runs `index.js`, it starts a process and the main thread is spun up, the event loop is started and the `app.listen()` function keeps it running waiting for inbound connections on the server. When the `/async` endpoint is hit, an event is emitted which triggers execution of the controller function. Being asynchronous it is deferred. When its turn in the queue reaches, it is invoked and our job is run asynchronously. The problem here is that since our application is running on a single thread, once the job starts running, the thread is occupied until it is done. I initially thought that this was what a blocking operation was but a look at the nodeJS docs would dispel that illusion. As stated before I presumed that asynchronous operations would always be deferred in favor of synchronous operations when they are available but a look at the execution order in the event loop would show that once a task starts, it has to finish before moving to the next instruction. This behavior is reasonable because once again we have just a single thread, the main thread. So if a single task decides to hog the CPU, there's not much the event loop can do for us.

> Blocking is when the execution of additional JavaScript in the Node.js process must wait until a non-JavaScript operation completes. This happens because the event loop is unable to continue running JavaScript while a blocking operation is occurring.
> 
> In Node.js, JavaScript that exhibits poor performance due to being CPU intensive rather than waiting on a non-JavaScript operation, such as I/O, isn't typically referred to as blocking. Synchronous methods in the Node.js standard library that use libuv are the most commonly used blocking operations. Native modules may also have blocking methods.
> 
> Source: [https://nodejs.org/en/learn/asynchronous-work/overview-of-blocking-vs-non-blocking](https://en.wikipedia.org/wiki/Computing)

This made it clear that this was a case of poor performance and CPU hogging.

## What’s the solution?

The only reason that this is a problem is because Javascript is single-threaded. What we need is one of two things. Either spin up a new thread or spin up a new process to run our job in. Luckily for us, nodeJS exposes core modules to do this. 

The nodeJS core module [\`child\_process\`](https://nodejs.org/api/child_process.html) allows us to spawn child Node processes with an event-driven IPC (Inter-Process Communication) messaging channel using `child_process.fork()` so child processes can share information with their parent process. 

While to create threads, we use the [\`worker\_threads\`](https://nodejs.org/api/worker_threads.html) nodeJS core module. The nodeJS documentation recommends using this module for handling CPU-intensive workloads which happens to be our issue. There is also the fact that it allows memory sharing. This is important to me because I would like to make use of the same database connection in both threads by passing the reference to it to the worker script. This would be impossible without a shared memory space.

> Workers (threads) are useful for performing CPU-intensive JavaScript operations..
> 
> Source: [https://nodejs.org/api/worker\_threads.html](https://en.wikipedia.org/wiki/Computing)

## Implementation

The patched code with multi-threading looks like this.

```javascript
// index.js
import { Worker } from 'node:worker_threads';
import { fileURLToPath } from 'url';
import path from 'path';
import express from "express"

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const app = express();
const pr = new Promise((resolve) => {
 return resolve()
})

const performDBOperation = () => {
 const largeNumber = 1_000_000_000_000;
 return largeNumber
}

const longRunningJob = (largeNumber) => {
 for (let i = 0; i <= largeNumber; i++) {
   if (i === largeNumber) {
     console.log(i)
   }
 }
}

app.get('/async-worker', async (req, res) => {
 console.log("hit");
 const data = await pr.then(() => performDBOperation());
 res.json({})
 pr.then(() => {
   // Create a worker thread by passing a the path to a script that will run the task
   const worker = new Worker(path.join(__dirname, 'worker.js'));
   // send the data returned from the DB to the worker thread
   worker.postMessage(data);

   // Register an event to handle when the task is done
   worker.on('message', (data) => {
     // Job Done - Perform clean up
     console.log('result - ', data);
   });
 })
})

app.listen(4000, () => {
 console.log("========================= App started ==============================")
})
```

I had to define `__filename` and `__dirname` because they are not exposed by default in ES modules. They are only available in CJS Node modules. `Worker` is imported from the `worker_threads` nodeJs core module. It accepts the path to the script that runs the job as an argument. We then post the data from the database to the worker thread and register a `message` event to carry out potential cleanup operations on the main thread (this is optional). 

Next up, we define the worker script.

```javascript
// worker.js
import { isMainThread, parentPort } from 'node:worker_threads';

const pr = new Promise((resolve) => {
 return resolve()
});

const longRunningJob = (largeNumber) => {
 for (let i = 0; i <= largeNumber; i++) {
   if (i === largeNumber) {
     console.log(i)
     return 'Job Done'
   }
 }
}

if (!isMainThread && parentPort) {
 console.log('Inside worker thread - ', isMainThread) // Should log false when endpoint is called
  parentPort.on('message', (data) => {
   console.log('long number', data) // data = 1_000_000_000
   pr.then(() => {
     // Perform task
     return longRunningJob(data)
   }).then((message) => {
     // alert the main thread that the job is done
     parentPort.postMessage(message)
   }).catch((error) => {
     // handle error
     console.error(error);
   })
 })
}
```

Here we import `parentPort` and `isMainThread` from the `worker_threads` module. The `parentPort` represents the main thread while the `isMainThread` is a boolean to check if we are on the main thread.

We want to make sure we only run this script if and only if we are not on the main thread so we don’t “block” it for lack of a better word. Once we are in the worker thread, we want to listen for the `message` event from the main thread. This is how we receive the data from the database required for the job to run. After that, we run the job asynchronously and return a message when it’s done by posting to the `message` event we registered on the main thread that handles the cleanup. We also handle errors.

Now we can run this code in the same manner as earlier described. You’ll notice that requests are no longer being delayed and the jobs run and post their results to the console when done. Problem fixed!

```bash
# You can run both commands in different terminal sessions for a better experience.
# & allows the command to run in the background
node index.js &
curl localhost:4000/async-worker
```

# Conclusion

Multi-threading is one way to implement concurrency in computer programs. NodeJS exposes the `worker_threads` module that can help us achieve this. So when next you have a CPU-intensive workload on a NodeJS application, you now know what to do. I’m pretty sure you’ll agree with me that waiting for long running tasks to be fulfilled is a pretty bad user experience.

That’s all folks! I hope you found this article useful. Drop your comments in the comments section if you have any queries or objections. It’ll be fun to interact with you guys.