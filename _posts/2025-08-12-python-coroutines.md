---
layout: post
title: >
  Notes on async-await
tags: [hobbies]
---

# Notes on async-await in Python

I have glanced through the subject of coroutines in python a few times already. It's inevitable that I'll forget what they are in the future, hence my hope is that this article will serve as a refresher to the future me (and may it be helpful to the distinguished reader in some capacity).

We are not going in depth. If the python async programming environment were a machine, today we are not dismantling the machine to take a look at the mechanics. Instead, we'll look at the buttons this machine exposes and what happens when you press these buttons.

## What's async programming?

**Is it the same as multithreading?** *Nope.*

It's easy to confuse asynchronous programming with multithreading (I did). They are both ways to get your computer to do a set of things faster. Maybe I can phrase that better. 

Let's say you have one job for the computer. Now your computer has the CPU, RAM, network and hard drive. Your job has parts where it is scheduled on the CPU and other parts where it is waiting for I/O from network or the hard drive. Now this job is happy because it's undisturbed by other jobs. There are times when the job is scheduled on the CPU - performing computations, while at other instances it's waiting for I/O. There isn't much we can do to reduce the turnaround time of this job without changing the core algorithm behind this job. 

> **Turnaround time** is the duration of time required for the job to finish.<br> 
> T<sub>turnaround</sub> = T<sub>finish</sub> - T<sub>start</sub>

Let's say there's a bundle, comprising of 10 such jobs, and you care about the turnaround time of this bundle. You still have access to one CPU core. What do you do? The naive way to do it is chain 10 blocking calls, one after the other. This is called synchronous programming. 

How do we do better? Perhaps multithreading crosses your mind. You launch 10 threads, each tasked with the completion of a job and the main thread waits for (technically, joins) these threads. Let's see what this looks like in code.

~~~python
import time
import threading

result = [0 for _ in range(10)]

def job(n):
    # CPU-bound simulation
    result[n] = sum(i * i for i in range(n * 1000))
    # I/O-bound simulation
    time.sleep(n)

threads = []

for i in range(10):
    t = threading.Thread(target=job, args=(i,))
    t.start()
    threads.append(t)

for t in threads:
    t.join()
~~~

This way, the responsibility of scheduling these threads is offloaded to the OS. To see a visualization of multithreading in Linux, take a look at [this post](2025-08-12-multithreading-visualization.md).

<span style="color:red">*TODO:*</span> what's wrong with multithreading? why async programming have a clear cut bridge.

Let us now take a look at the paradigm of asynchronous programming. I say this is a programming paradigm because it's a fundamental shift in the mental model of the programmer's approach to solving the forementioned problem. A central idea in asynchronous programming is the existence of the event loop - I think of it as a while loop placed on top of a queue. The programmer can submit a job to the queue, and it is the event loop's responsibility to see all the jobs in its jurisdiction to completion by scheduling and preempting them as required.

## Python coroutines

Let's understand asynchronous programming with an example - the same problem as before.

~~~python
import asyncio

async def job(n: int) -> int:
    # CPU-bound simulation
    acc = sum(i * i for i in range(n * 1000))
    # I/O-bound simulation
    await asyncio.sleep(n)  
    return acc

async def main():
    tasks = [asyncio.create_task(job(i)) for i in range(10)] 
    results = await asyncio.gather(*tasks)
    print(results)

if __name__ == "__main__":
    asyncio.run(main())
~~~

The following concepts will help us wrap our heads around this code snippet:
- Futures
- asyncio library and its APIs
- Language features: async-await

`asyncio` is the library that provides the machinery for the event loop we discussed earlier. Before diving into asyncio, let's first take a look at what futures are.

~~~python
# Synchronous programming
result = start_and_finish_job()

# Asynchronous programming
result_future = started_but_unfinished_job()
~~~


**`futures` lie at the core of asynchronous programming.**

When we are programming synchronously, the return value of a function call usually contains the result which we are free to use just after the function call - there are no two ways about it. We call this a blocking call.

In the async world, the caller has an option to tell the language to start working on a job, work on it in the background and let them know if the job is done when asked. 

Say you order a pizza at a restaurant and they hand you a box, and ask you to take a seat at a table nearby - strange! This is a high-tech restaurant and they say you need not come over to the counter to collect your order, the pizza will materialize inside your box when it's ready!!

You marvel at the rate technology is progressing and you get busy chatting with your dining companion, your friend or your laptop, depending on your social circle :)

After 10 mins, you remember about the pizza and check the box by lifting the lid, it's not yet ready. It's alright - you get back to your interesting conversation. 5 more mins pass and you are hungry. This time, you keep the lid open because you have run out of topics to discuss and want to start eating as soon as the pizza arrives, and it appears in a couple of minutes. You are happy, and you once again can't help chuckling at rate of progress of technology.

That box is the `future`. It's a container for job that has started and is being worked on, but may not be finished yet.

~~~python
function orderPizza():
    # Start the pizza-making job in the background
    futurePizza = startBackgroundJob(makePizza)
    return futurePizza


function main():
    # Step 1: Place the order
    pizzaBox = orderPizza()   // returns a Future object

    # Step 2: Chat with your friend/laptop while pizza is cooking
    chatForMinutes(10)

    # Step 3: Check the box
    if pizzaBox.isReady():
        eat(pizzaBox.get())
    else:
        print("Not ready yet... back to chatting")
        chatForMinutes(5)

    # Step 4: Decide to 'wait' for it now
    pizza = pizzaBox.waitUntilReady()  // blocking until pizza is done
    eat(pizza)


function makePizza():
    # simulate cooking time
    waitMinutes(12)
    return "Delicious Pizza"


# ---- Run the scenario ----
main()
~~~

This syntax is a bit tedious, so python makes it easy to do the same with syntactic sugar. In fact python does this so well, that we need not think of futures at all!

There's a library called `asyncio` which provides the engine - the event loop which executes the asynchronous functions, which are called coroutines, by the way. Let's take a look at a coroutine we saw earlier:

~~~python
async def job(n: int) -> int:
    ...
    return result
~~~

`async def` is a way of signalling that this function houses code that can be done in the background. It says to the caller - you need not wait for me to complete, go do your thing while I get your *order* ready.

Calling `job()` does not start the job. In fact, it returns a *coroutine* - a function that can be paused and resumed. 

To actually execute the coroutine, you need to hand it over to an executor. asyncio provides an event loop and APIs through which one can run jobs on the event loop.

~~~python
import asyncio

async def main():
    print('hello')
    await asyncio.sleep(1)
    print('world')

asyncio.run(main())
~~~

Here the main() coroutine is scheduled on the event loop using `asyncio.run`.

What's `await`? It awaits the completion of an *awaitable*, which can be either one of the following:
1. a future - an object that holds an incomplete result
2. a coroutine - a function that can be paused or resumed
3. a task - a wrapper over coroutine (inheriting from future) that is scheduled on the event loop

await can only be placed inside a coroutine (an async def function). Why? Because await tell the interpreter that this coroutine would like to suspend execution ([yield from](https://lerner.co.il/2020/05/08/making-sense-of-generators-coroutines-and-yield-from-in-python/)) until the awaitable is complete and the control is transferred to the event loop. When the awaitable is complete, the coroutine housing the await is rescheduled on the event loop.

Quiz:

Can you predict the last line of output of the following two code snippets?


~~~python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    print(f"started at {time.strftime('%X')}")

    await say_after(1, 'hello')
    await say_after(2, 'world')

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
~~~

```plaintext
started at 17:13:52
hello
world
```
<details>
    <summary>What comes next?</summary>
    finished at 17:13:55
</details>


~~~python
async def main():
    task1 = asyncio.create_task(
        say_after(1, 'hello'))

    task2 = asyncio.create_task(
        say_after(2, 'world'))

    print(f"started at {time.strftime('%X')}")

    # Wait until both tasks are completed (should take
    # around 2 seconds.)
    await task1
    await task2

    print(f"finished at {time.strftime('%X')}")
~~~

~~~plaintext
started at 17:14:32
hello
world
~~~

<details>
    <summary>What comes next?</summary>
    finished at 17:14:34
</details>


~~~python
import asyncio


async def T1():
    print("T1 start")
    await asyncio.sleep(2)
    print("T1 end")

async def T2():
    print("T2 start")
    print("T2 end")

async def T3():
    print("T3 start")
    print("T3 end")

async def main():
    print("main start")
    t1 = asyncio.create_task(T1())
    t3 = asyncio.create_task(T3())
    t2 = asyncio.create_task(T2())
    print("main middle")
    await t1
    await t2
    await t3
    print("main end")

if __name__ == "__main__":
    asyncio.run(main())
~~~

<!-- in your Markdown post -->
<div class="video">
  <iframe
    src="https://www.youtube-nocookie.com/embed/3iGY4tb2dXw"
    title="YouTube video"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowfullscreen
  ></iframe>
</div>
