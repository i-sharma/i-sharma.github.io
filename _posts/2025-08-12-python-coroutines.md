---
layout: post
title: >
  Notes on async-await
tags: [hobbies]
---

# Notes on async-await in Python

I've glanced through coroutines in Python a few times already. It's inevitable I'll forget the details, so this note is a refresher for future-me (and, in some capacity, useful to a distinguished passerby).

We're not taking apart the engine today. If the Python async world were a machine, we're just pressing the buttons to see what lights up.

---

* TOC
{:toc}

---

## 1) What is async programming?

**Is it the same as multithreading?** *Nope.*

It's easy to confuse async with threads (I did). They both try to reduce *wall-clock* time, but they do it differently.

Think about a single job. It alternates between **CPU time** (compute) and **I/O wait** (disk/network). With one job, you usually can't beat its turnaround time without changing the algorithm.

> **Turnaround time** = time to finish the job.  
> T<sub>turnaround</sub> = T<sub>finish</sub> − T<sub>start</sub>

Now imagine **10 independent jobs** on **one CPU core**.

- **Synchronous (naïve):** call them one after another (10 blocking calls).
- **Multithreading:** start 10 threads; while some threads are waiting on I/O, others can run.

### Threads example

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

Here, the OS scheduler decides which thread runs.  
*Footnote for Python folks:* due to the **GIL**, only one thread executes Python bytecode at a time in CPython. Threads still help when you spend time waiting on I/O; they don't speed up CPU-bound Python code. (For CPU-bound work, think **multprocessing** or native extensions.)

**Where threads bite (and why async exists):**
- Shared-state complexity (locks, races, heisenbugs).
- Context-switch overhead when you scale out.
- In CPython, limited wins for CPU-bound work.
- Hard to express "do these 200 I/O things, then continue" without a small forest of threads.

Async tackles the same class of problems (lots of I/O) with a different **control-flow style**.

---

## 2) Event loop: the mental model

Picture an **event loop**: a loop that drives a queue of tasks. Tasks run until they **await** something, then politely **yield**. When that thing is ready, the loop resumes them right after the `await`. This is **cooperative** scheduling (no preemption by the loop itself).


> Threads are like trying to hold ten phone conversations on ten different handsets at once; async is putting each call on speaker, hitting "mute," and letting the callers shout "I'm back!" when they need your attention.

---

## 3) Coroutines with `asyncio`

Same 10-job scenario, async-style:

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

Things to keep in your head:
- **Futures**
- **`asyncio`** and a few core APIs
- Language constructs: **`async` / `await`**

`asyncio` provides the event loop and the primitives to schedule / pause / resume work.

---

## 4) Futures (or: the pizza box)

~~~python
# Synchronous programming
result = start_and_finish_job()

# Asynchronous programming
result_future = started_but_unfinished_job()
~~~

A **future** is a container for a result that will exist *later*.

Imagine a very advanced pizzeria: you place an order and receive a **box** immediately. The pizza will "materialize" in the box when it's ready. You chat, you peek, you wait only when you actually want to eat. The box is the **future**: the job is underway; you'll collect the result when it's done.

Python smooths this so well that most of the time you don't think about futures directly.

---

## 5) `async` / `await` in practice

`async def` marks a function as a **coroutine**. When you call it, nothing runs yet-you get a *coroutine object* (something that can be paused and resumed).

To actually execute it, either **`await`** the coroutine inside another coroutine, or wrap it in a **Task** with `asyncio.create_task()` and await the task later. The **event loop** drives it, resuming right after each `await`.

"Background" here means *non-blocking for you*: the coroutine yields control at `await` so other coroutines can run. It's still usually the **same thread** unless you explicitly offload blocking work to a thread/process pool via `asyncio.to_thread()` or `loop.run_in_executor()`.

~~~python
import asyncio

async def main():
    print('hello')
    await asyncio.sleep(1)
    print('world')

asyncio.run(main())
~~~

`asyncio.run` creates the loop, runs `main()` to completion, then cleans up.

**What does `await` do (really)?**  


It **suspends** the current coroutine until an **awaitable** completes, handing control back to the loop. Awaitables include:

1. **Future** - an object representing an incomplete result  
2. **Coroutine object** - produced by calling an `async def` function  
3. **Task** - a Future subclass that wraps/schedules a coroutine on the loop

When the awaitable finishes, the loop **resumes** your coroutine *right after* the `await`. (Under the covers, this is very much like `yield from` with a fancy hat.)

---

## 6) Task ordering (a tiny subtlety)

```python
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
```

`create_task` schedules coroutines to start soon; you often see the **first** slice of each run in the order you created them. After a task hits an `await`, though, the loop resumes whichever task becomes ready next (timers, I/O). Your `await t1; await t2; await t3` only says when **`main`** waits on them, not how they interleave internally.

---

<div class="video">
  <iframe
    src="https://www.youtube-nocookie.com/embed/3iGY4tb2dXw"
    title="YouTube video"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowfullscreen
  ></iframe>
</div>

---

## 7) Quiz: sequential vs concurrent awaits

**Sequential awaits**

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

**Output:**
```plaintext
started at 17:13:52
hello
world
````

<details>
  <summary>What comes next?</summary>
  finished at 17:13:55
</details>

---

**Concurrent with tasks**

```python
async def main():
    task1 = asyncio.create_task(say_after(1, 'hello'))
    task2 = asyncio.create_task(say_after(2, 'world'))

    print(f"started at {time.strftime('%X')}")
    await task1
    await task2
    print(f"finished at {time.strftime('%X')}")
```

**Output:**
```plaintext
started at 17:14:32
hello
world
```

<details>
  <summary>What comes next?</summary>
  finished at 17:14:34 <br>
  The second finishes in ~2s instead of ~3s because the sleeps overlap.
</details>

---

## 8) Summary

You don't need to master every asyncio API right now. Just remember:
- `async def` creates a coroutine
- `await` pauses for something else to finish
- The *event loop* is the traffic cop keeping it all moving

The rest you'll pick up the next time you forget. Or just do what I did - ask your favourite LLM :)