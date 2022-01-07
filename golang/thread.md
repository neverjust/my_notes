## goroutine 是什么

goroutine建立在操作系统线程基础之上，与线程之间实现了一个**多对多**的两级线程模型，内核调度线程，线程又负责对gorountine进行调度。goroutine和线程的区别：

**内存消耗更少：**

Goroutine所需要的内存通常只有2kb，而线程则需要1Mb（500倍）

**创建与销毁的开销更小：**

由于线程创建时需要向操作系统申请资源，并且在销毁时将资源归还，因此它的创建和销毁的开销比较大。相比之下，goroutine的创建和销毁是由go语言在运行时自己管理的，因此开销更低。

**切换开销更小：**

只是goroutine之于线程的主要区别，也是golang能够实现高并发的主要原因。线程的调度方式是抢占式的，如果一个线程的执行时间超过了分配给它的时间片，就会被其他可执行的线程抢占。在线程切换的过程中需要保存/恢复所有的寄存器信息，比如16个通用寄存器，PC（Program Counter）、SP（Stack Pointer）段寄存器等等。而goroutine的调度是协同式（线程执行完后，主动通知系统切换）的，它不会直接地与操作系统内核打交道。当goroutine进行切换的时候，之后很少量的寄存器需要保存和恢复（PC和SP）。因此goroutine的切换效率更高。

因为是协同式的调度，所以需要golang维护调度切换：

- Channel接收或者发送会造成阻塞的消息
- 当一个新的goroutine被创建时
- 可以造成阻塞的系统调用，如文件和网络操作
- 垃圾回收

调度器具体是如何工作的呢，Golang调度器中有三个概念：

- Processor（P）
- OSThread（M）
- Goroutines（G）

在一个Go程序中，可用的线程数是通过GOMAXPROCS来设置的，默认值是可用的CPU核数。我们可以用runtime包来动态改变这个值。OSThread调度在processor上，goroutines调度在OSThreads上。

Golang的调度器可以利用多processor资源，在任意时刻，M个goroutine需要被调度到N个OS threads上，同时这些threads运行在至多GOMAXPROCS个processor上（N <= GOMAXPROCS）。Go scheduler将可运行的goroutines分配到多个运行在一个或多个processor上的OS threads上。

每个processor有一个本地goroutine队列。同时有一个全局的goroutine队列。每个OSThread都会被分配给一个processor。最多只能有GOMAXPROCS个processor，每个processor同时只能执行一个OSThread。Scheculer可以根据需要创建OSThread。

在每一轮调度中，scheduler找到一个可以运行的goroutine并执行直到其被阻塞。由此可见，操作系统的一个线程下可以并发执行上千个goroutine，每个goroutine所占用的资源和切换开销都很小，因此，goroutine是golang适合高并发场景的重要原因。

