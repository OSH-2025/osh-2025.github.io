# 选做部分

## 使用线程池机制（10%）

每次创建新线程需要额外的开销，请自行思考设计，是否可以预先创建一组线程，在请求到达时分配给其中一个，以减少创建的额外开销。

参考资料：

1. pthread 即 POSIX 标准线程库（POSIX Threads），是一个用于多线程编程的跨平台 C 语言库。它提供了一组 API，用于创建和管理线程，以及实现线程间的同步和通信。
2. thread 库是 C++ 标准库的一部分，从 C++11 开始引入，提供了对多线程编程的支持。它封装了底层的线程实现（如 POSIX pthread），为开发者提供了更高层次、更易用的线程管理接口

- POSIX thread (pthread) libraries：<https://www.cs.cmu.edu/afs/cs/academic/class/15492-f07/www/pthreads.html>
- thread libraries: <https://en.cppreference.com/w/cpp/thread/thread>

## 使用缓存机制（10%）

对于经常访问的资源，可以使用内存缓存或预取对资源读取进行加速。一个常见的例子是，html 文件中包含的图片，Javascript 代码，CSS 代码等具有显著的访问模式，可以设计机制对这类特定的访问顺序进行缓存或预取，使得响应速度更快。

注意：

- 多线程配合缓存机制需要注意共享数据访问时需要加锁；
- 测试过程中可能会删除修改文件，需要设计缓存的更新机制；
- Linux 本身对文件有透明的缓存机制，请在报告中分析自己的缓存机制带来的性能提升。

## 使用 I/O 复用或异步 I/O（20%）

除了多线程，I/O 复用与异步 I/O 技术也是实现并发服务器的常见方式。在示例程序中，我们可以发现 `recv` 大部分时间都在等待数据的到来，实际上处于空闲状态。因此在这种情况下，多线程实际上是没有必要的，我们可以使用非阻塞 I/O 与 I/O 复用技术实现相同的功能。常见的 I/O 复用实现有：`select`、`epoll`，异步实现有 async/await 与 `io_uring`。

本选做要求从多种技术中任选一种完成。在本项选做中，**不得**使用多线程技术。如果选择该选做，**你需要同时提交多线程版本和该选做版本的代码**，并在 `README.md` 文件中说明选做代码的编译运行方法和性能分析。两个版本都会进行正确性测试（选做部分的正确性会影响选做部分的得分）。

### I/O 复用

Linux 提供 `select`、`epoll` 等高级的 I/O 复用模式，请自行了解相关概念，使用 `select` 或 `epoll` 等提高服务器并发性能。相关文档请查阅互联网，或参考 2022 年的双人聊天室示例代码：<https://osh-2022.github.io/lab3/#select-4>。

### async/await

协程是代码控制（而不是 OS 调度控制）的线程。这里的代码不是指你自己编写的代码，而是编译运行系统为你的代码提供的最外层的 runtime。不同的 runtime 决定协程的不同调度策略，包含协程应该由多少线程执行、应该先执行哪些协程。因此，协程可以与线程**不**一一对应。

对协程的理解：`async fn f()` 将函数 `f` 声明为了一个特殊的异步函数，进行 `f()` 调用时不会立即执行函数内容，而是返回一个 `Coroutine/Future/Promise` 对象，例如可以 `let a = f()`；对这个对象执行 `a.await` 会在运行到此处时触发 runtime 的调度机制，runtime 可以选择继续执行 `f()`，也可以去执行其他的 `.await` 点位。这里的 `await` 点位相当于一个 breakpoint，runtime 保证在执行完 `f()` 的内容后再继续执行 `a.await` 之后的代码。

协程有多种实现方式，例如有栈协程和无栈协程。无栈协程的实现方式是通过编译器将协程的状态保存在一个结构体中，然后通过状态机来实现协程的切换。而有栈协程则是通过编译器将协程的状态保存在栈中，然后通过切换调用栈来实现协程的切换。无栈协程效率更高，但实现更为麻烦，因此主流编程语言两种实现均存在。

足够现代的编程语言中几乎都有 async/await 的实现，包括：

- Python，从 3.5 开始正式叫 async/await；
- Javascript，Promise，从 ECMAScript 2017 开始标准化；
- Rust，Future trait，从 1.39.0 开始；
- Kotlin，structured coroutine，主打点之一/对比 Java 的优势之一；
- C++，coroutine，从 C++20 开始。

你可以尝试将自己的服务器改写为用 async/await 实现。一般来说，只要你的几个工作函数是 `async fn` 即可。如果你不确定自己的实现是否「足够 async/await」，请积极询问助教们。

C++ 的 coroutine 支持在 C++20 才加入且可能不够完全，资料较少，我们建议在实现此部分内容时使用 Rust 完成，但如果你愿意挑战 C/C++ 下的协程编程也是可行的。

仅在此章节中，我们允许使用热门的 Rust 异步编程库，例如 tokio 和 async-std；以及 C++ 的 Boost.Asio 库。如果你想使用其他的异步库亦可提前询问我们。

参考资料：

- 协程概念说明：<https://www.baeldung.com/java-threading-models>；
- Rust tokio 库文档：<https://docs.rs/tokio/latest/tokio/>。

### io_uring

`io_uring` 是 Linux 5.1 引入的一种新的 I/O 模型，它的目标是取代 `epoll`，并且在性能上有很大的提升。`io_uring` 的设计思想是将 I/O 操作与 I/O 完成事件分离，从而避免了 `epoll` 的回调机制。`io_uring` 的使用方法与 `epoll` 类似，但是它的性能更高，因为它不需要回调，也不需要用户态与内核态之间的切换。`io_uring` 的详细说明请参考：<https://kernel.dk/io_uring.pdf>。
