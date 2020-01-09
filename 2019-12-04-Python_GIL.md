# 简介：
GIL（Global Interpreter Lock）即“解释器全局锁”，它就是一个 mutex（或叫 lock），每一个线程想要获取 Python 解释器来运行 Python 代码（二进制码）时都需要获得该锁，这样就会导致在任何时间点仅会有一个线程位于运行状态（其他线程都在 require lock 处阻塞）。

# 为什么 Python 需要GIL？
原因：Python 使用的是引用计数（reference count）来进行内存管理（memory management），如果没有GIL，那么可能会在引用计数计算的时候产生竞争（race condition），这会导致 bug。

再问：为什么不给每一个对象加锁，而是选择了直接来一手解释器全局锁这种操作呢？<br>
答：给每一个对象加锁，锁过多，也会有死锁（dead lock）的问题，且每个对象一个锁，开销太大，也会带来性能问题。

不只是 Python，Ruby 也使用了 GIL。但是引用计数不是进行内存管理的唯一策略，有很多其他语言使用了其他的线程安全的机制进行内存管理（垃圾回收）如 JAVA 的跟踪收集器（Tracing Collector）。

问：为什么当时 Python 选择了 GIL 呢？<br>
答：
1. 因为Python当时出来的时候很早，还没有多线程的概念。
2. 全局锁，实现简单。
3. 很多现存的 C 库也能很方便的作为 Python 扩展集成到 CPython 解释器。

# GIL产生的影响
因为一个时间点只能有一个线程位于运行状态，故而即使是多线程，也无法利用多核优势并行运行。对 CPU 密集型任务影响很大，但是对于 IO 密集型任务影响不大，因为线程主要在等待 IO。

# 对策
1. 使用多进程
2. 使用其他的 Python 解释器如 PyPy。
3. 使用别的语言（逃

# 参考：
- [What is the Python Global Interpreter Lock (GIL)?-REAL PYTHON](https://realpython.com/python-gil/)
