# 一些概念：

## Concurrency and parallelism：

parallelism，并行，是物理上的同时运行。

concurrency，并发，是逻辑上的同时运行。

并行是并发的子集，并行的任务一定是并发的，并发的任务不一定是并行的。

## 并发的方法有：

1. 多进程（multi-processing）
2. 多线程（multi-threading）
3. 异步IO（Asynchronous IO）

## 异步IO（Asynchronous IO）

Asynchronous IO (async IO) 是一种和编程语言无关的编程范式，是单进程单线程的条件下实现任务并发的一种方式，实现的方式是使用一个Event Loop来跑很多的协程（coroutine），每一个coroutine在进行阻塞函数调用（如读取磁盘文件，等待网络数据传输，读取数据库）的时候将执行权交给其他协程，等到阻塞的任务完成，再重新获得执行权继续往下执行。

很多语言都支持异步，如JavaScript，Python。

## Python中实现并发该如何选择：

多线程：

注意在Python中由于有GIL的存在，导致程序即使是多线程的，也只能使用一个CPU核心来跑，所以如果将多线程用于CPU密集型任务，不仅不会取得好的优化，反而会因为锁及线程切换的开销导致性能下降。但是用在IO密集型任务里是完全没问题的，因为一个线程阻塞了会挂起，就可以跑另一个线程了。

所以在Python中我们通常将多进程用于CPU密集型任务，多线程用于IO密集型任务。

当然也可以将两者结合起来，多进程 + 多线程。

异步IO：

异步IO和多线程类似，用于IO密集型任务的并发，与多线程相比的优点是：

- 因为是单线程，所以就没有竞争（race condition）的问题，故不需要加锁，因而可以避免竞争可能带来的诸多难以追踪的bug。理解上也更符合人类的直觉？
- 更轻：要同时进行成千上万个任务，如果创建成千上万个线程，显然很多机器都无法做到，但是创建成千上万个async IO task是一点问题也没有的。且线程是系统级别的，执行权由系统负责切换，协程中执行权是程序自己代码控制 + Event Loop负责的。

异步IO的注意点：

**异步的话，必须所有的阻塞操作都要异步。**因为在协程中调用synchronous function还是阻塞整个线程，并不会把控制权交给其他的协程。所以在考虑“到底是使用异步IO还是使用多线程？”的时候，要考虑到用到的库是不是支持async。

# Python中的async

- `async/await`关键字：

由`async def`定义的函数是一个协程，调用后会返回一个coroutine对象，这个coroutine对象必须使用Event Loop来跑或被其他协程await调用才能真正运行，如下：

    >>> async def foo():
    ...     print('Hello Async')
    ...
    >>> foo()
    <coroutine object foo at 0x7f92518b90e0>
    >>> import asyncio
    >>> asyncio.run(foo())
    Hello Async

`await <awaitable object>`：await语句会暂停当前函数的运行并把控制权交给Event Loop，直到await等待的任务完成，Event Loop会将控制权转移回来，函数才会继续向下执行。

- 标准库中的`asyncio` package。

示例程序：

    #!/usr/bin/env python3
    # async.py
    
    import asyncio
    
    async def count():
        print("One")
        await asyncio.sleep(1)
        print("Two")
    
    async def main():
        await asyncio.gather(count(), count(), count()) # 将多个协程chain起来。
    
    asyncio.run(main())

一些限制：

- 由`async def`定义的协程函数体内可以使用`await`，`return`，`yield`，但不能使用`yield from`。（如果使用了yield，那么该函数调用后会返回一个**asynchronous generator**，我们应当使用`async for`来迭代它）
- `await`必须只能在`async def`定义的协程函数体中使用。
- `await`的对象必须是一个**awaitable object**。awaitable object即为定义了__await__方法的对象，__await__方法应该返回一个iterator。coroutine object is awaitable。

- `async for`：

我们要知道的就是**asynchronous iterable object**（包括**asynchronous iterator**和**asynchronous generator**）允许我们在这个object内部调用异步方法，这是普通**iterable object**（包括普通iterator和普通generator）做不到的，而我们必须使用`async for`语句来迭代它而不能使用一般`for`语句。

即普通**iterable object**使用`for`语句来迭代，而**asynchronous iterable object**应使用`async for`语句来迭代。

- `async with`：

我们要知道的就是**asynchronous context manager**允许我们在enter和exit的代码块中调用异步函数，这是普通**context manager**做不到的，而我们必须使用`async with`语句来使用它而不能使用一般`with`语句。

即普通**context manager**使用`with`语句来使用，**asynchronous context manager**应使用`async with`语句来使用。

# 高级语法

## asynchronous generator：

由PEP-525引入，其目标与解决的问题文章中说的很清楚：

> Regular *generators* (introduced in PEP 255) enabled an elegant way of writing complex data producers and have them behave like an iterator.
However, currently there is no equivalent concept for the asynchronous iteration protocol (*async for*). This makes writing asynchronous data producers unnecessarily complex, as one must define a class that implements __aiter__ and __anext__ to be able to use it in an *async for* statement.

和generator一样，在`def`定义的普通函数体中使用`yield`语句，使得该函数变为一个generator，调用后返回一个generator object，可以通过`for`语句来迭代。

而在`async def`定义的异步函数体中使用`yield`语句，使得该函数变为一个asynchronous generator，调用后返回一个asynchronous generator，可以使用`async for`语句来迭代。

代码示例：

    async def ticker(delay, to):
        """Yield numbers from 0 to `to` every `delay` seconds."""
        for i in range(to):
            yield i
            await asyncio.sleep(delay)
    
    >>> ticker()
    <async_generator object bar at 0x7f92518b93b0>

## Asynchronous Comprehensions

由PEP 530引入，和普通的comprehension解决的问题一样。

引入语法糖，使用`async for`语句生成list和dict。

普通comprehension迭代iterable object，Asynchronous Comprehension迭代asynchronous iterable object。

# 自己的理解：

- python中的coroutine其实是靠generator实现的，注意结合起来方便理解。

# 具体细节：

## async for与**asynchronous iterable**

见：[PEP492-Coroutines with async and await syntax-asynchronous-iterators-and-async-for](https://www.python.org/dev/peps/pep-0492/#asynchronous-iterators-and-async-for)一节：

> An **asynchronous iterable** is able to call asynchronous code in its `iter` implementation, and **asynchronous iterator** can call asynchronous code in its `next` method. To support asynchronous iteration（协议）:
1. An object must implement an method *__aiter__* returning an *asynchronous iterator object*.
2. An *asynchronous iterator object* must implement an *__anext__* method returning an *awaitable*.
3. To stop iteration __anext__ must raise a *StopAsyncIteration* exception.

An example of asynchronous iterable:

    class AsyncIterable:
        def __aiter__(self):
            return self
    
        async def __anext__(self):
            data = await self.fetch_data()
            if data:
                return data
            else:
                raise StopAsyncIteration
    
        async def fetch_data(self):
            ...

## async with与**asynchronous context manager**

见：[PEP492-Coroutines with async and await syntax-asynchronous-context-managers-and-async-with](https://www.python.org/dev/peps/pep-0492/#asynchronous-context-managers-and-async-with)一节

> An **asynchronous context manager** is a context manager that is able to suspend execution in its *enter* and *exit* methods.
To make this possible, a new protocol for asynchronous context managers is proposed. Two new magic methods are added: *__aente__* and *__aexit__*. Both must return an *awaitable*.

An example of an asynchronous context manager:

    class AsyncContextManager:
        async def __aenter__(self):
            await log('entering context')
    
        async def __aexit__(self, exc_type, exc, tb):
            await log('exiting context')

# 参考：

教程：

- [Python 异步编程入门-阮一峰](http://www.ruanyifeng.com/blog/2019/11/python-asyncio.html)
- [Async IO in Python: A Complete Walkthrough](https://realpython.com/async-io-python/)

PEP：

- [PEP 492 -- Coroutines with async and await syntax](https://www.python.org/dev/peps/pep-0492/)
- [PEP 525 -- Asynchronous Generators](https://www.python.org/dev/peps/pep-0525/)
- [PEP 530 -- Asynchronous Comprehensions](https://www.python.org/dev/peps/pep-0530/)
