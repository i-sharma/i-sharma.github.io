---
layout: post
title: >
  RAII in Python
tags: [tech-non-ai]
---

# RAII in Python

Resource management is a recurring problem in programming. Let's start with understanding how C++ manages resources.

## RAII in C++

In C++, the solution to the resource management problem is [RAII: *Resource Acquisition Is Initialization*](https://en.cppreference.com/w/cpp/language/raii.html). The idea is simple: when an object is created, it acquires a resource (a file, a lock, a socket). When the object is destroyed, it automatically releases the resource in its destructor.

~~~cpp
#include <fstream>
#include <string>

void readFile() {
    std::ifstream f("data.txt");
    std::string line;
    while (std::getline(f, line)) {
        // process line
    } // f goes out of scope, file automatically closed
}
~~~

The destructor call (`~ifstream`) ensures the file is closed. No `close()` call needed.

RAII in C++ isn’t just about files. It generalizes to any resource: memory, locks, sockets.  
That’s why smart pointers (`std::unique_ptr`, `std::shared_ptr`) exist. They wrap raw pointers and ensure memory is freed automatically, even if you forget to call `delete`.  

- `unique_ptr`: sole owner of a resource, auto-deletes when it goes out of scope.  
- `shared_ptr`: shared ownership, deletes only when the last reference is gone.  

This eliminates a huge class of bugs (memory leaks, double deletes).


The real strength of RAII shows up with **exceptions**.  
If an exception is thrown, destructors still run. Files close, memory frees, locks release without extra code.  
This is C++’s answer to "finally" blocks in other languages.

---

## RAII in Python: `with`

Python doesn’t have deterministic destructors like C++, but it gives us context managers: `__enter__` and `__exit__`. They wrap up the same idea: resource setup and teardown, just scoped to a `with` block.

~~~python
class FileWrapper:
    def __init__(self, path):
        self.path = path

    def __enter__(self):
        self.f = open(self.path, "r")
        return self.f

    def __exit__(self, exc_type, exc_value, traceback):
        self.f.close()

# usage
with FileWrapper("data.txt") as f:
    for line in f:
        # process line
~~~

Here `__enter__` acquires the resource, and `__exit__` ensures cleanup, even if exceptions occur inside the block.

Of course, you don’t usually write this yourself. Python’s built-ins (`open`, `threading.Lock`, etc.) already implement it:

~~~python
with open("data.txt") as f:
    data = f.read()
# file is closed automatically
~~~

Like C++, Python’s context managers guarantee cleanup even if exceptions happen.  
No matter how the block exits, normal return or an error, `__exit__` runs, just like a `finally` block.

---

## Async RAII: `async with`

When resources are asynchronous (say, a database connection pool or a websocket), Python extends the same model with `__aenter__` and `__aexit__`.

~~~python
class AsyncConnection:
    async def __aenter__(self):
        self.conn = await connect()
        return self.conn

    async def __aexit__(self, exc_type, exc_value, traceback):
        await self.conn.close()

# usage
async def main():
    async with AsyncConnection() as conn:
        await conn.query("SELECT * FROM users")
~~~

The guarantee is the same: the resource is released when the block ends, regardless of exceptions.

Async context managers extend the same guarantee to asynchronous code: resources get released on both normal exit and exceptions.  


---

## Takeaway

- **C++ RAII**: tie resource lifetime to object lifetime.
- **Python RAII**: use `with` / `async with` to tie resource lifetime to a block.

Both approaches ensure you don’t forget cleanup.

This is one of those patterns that, once you understand it, you’ll see it everywhere, from file handles to network sockets to database connections.