# I/O软件原理

## I/O 软件目标 Goals of the I/O Software

- **设备独立性（device independent）**：即它可以访问任意IO设备，而无需事先事先指定设备。

- **统一命名（uniform naming）**：一个文件或一个设备的名字应该是一个简单的字符串或者一个整数，它不应该依赖于设备。**用这种方法，所有文件和设备都采用相同的方式——路径名进行寻址。**

- **错误处理（error handling）**：错误应该尽可能地在接近硬件的层面得到处理。当控制器发现问题是，首先它自己处理，不行再给设备驱动程序处理。只有在底层软件处理不了的情况下，才将错误上交高层处理。

- **同步（synchronous）（即阻塞 blocking）和异步（asynchronous）（即中断驱动 interrupt-driven）传输**：多数物理IO是异步的——CPU启动传输之后便转去做其他的工作，直到中断发生。正是操作系统使实际上的中断驱动的操作变为在用户程序看了是阻塞式的操作。

- **缓冲（buffering）**：数据离开一个设备后通常并不能直接存放到其最终的目的地。例如，从网络上进来一个数据包，直到将该数据包存放在某个地方并对其进行**检查**，操作系统才直到要将其放在何处。**此外，某些设备具有严格的实时约束（real time contraints）（例如，数字音频设备），所以数据必须预先放置在输出缓冲区之中，从而消除缓冲区填满速率和缓冲区清空速率之间的相互影响，以避免缓冲区欠载(buffer underruns)。缓冲涉及大量的复制工作，并且经常对IO性能有重大影响。**
- **共享设备和独占设备（shared vs dedicated devices）**:有些IO设备可以同时（at the same time）被多个用户使用，如磁盘，但是磁带就只能被用户独占使用。共享和独占设备还会对死锁产生影响。

## 程序控制I/O Programmed I/O

I/O 可以有三种不同的方式实现，本节先记录程序控制IO，**IO的最简单形式是让CPU做全部工作，这一方法称为程序控制I/O(Programmed I/O)。**

以例子说明程序控制IO是最简单的，考虑一个用户进程，该进程想在打印机上打印8个字符，有以下过程

![print String](https://blog-1300663127.cos.ap-shanghai.myqcloud.com/BackEnd_Notes/operating%20system/printString.png)

- 在用户空间的一个**缓冲区组装字符串**
- 用户进行**发出系统调用**打开打印机来获得打印机以便进行写操作
- 操作系统通常将字符串**复制到内核空间**中的一个数组中，在这里访问更加容易（因为内核可能必须修改内存映射才能到达用户空间）
- 操作系统查看打印机当前是否可用，如果不可用就等待直到它可用，一旦打印机可用，**操作系统就复制第一个字符到打印机的寄存器中，这一操作激活打印机，打印机打印第一个字符。**在这个例子中使用了**内存映射I/O(memory-mapped I/O)**，这个过程中操作系统进入了一个密闭的循环，一次输出一个字符。
- 直到整个字符串打印完，控制返回到用户程序

我们可以看到**程序控制IO的最根本的方面：输出一个字符后，CPU要不断查询设备以了解它是否就绪准备接收另一个字符。这以行为经常称为轮询（polling）或忙等待（busy waiting）。**

程序控制IO 十分简单但是有缺点：**直到全部IO完成之前要占用CPU的全部时间，当CPU有很多事情要做时，忙等待是低效的。**

## 中断驱动I/O Interrupt-Driven I/O

我们假设在不缓冲字符而是在每个字符到来时便打印的情形，如果打印机打印每个字符需要10ms，**使用程序控制IO时意味着，当每个字符写到打印机的数据寄存器中之后，CPU将有10ms搁置在无价值的循环中，等待允许输出下一个字节。**

**而允许CPU在等待打印机变为就绪的同时做某些事情的方式就是使用中断。当打印字符串的系统调用被发出时，字符串缓冲区被复制到内核空间，并且一旦打印机准备好接收一个字符时就将第一个字符复制到打印机中，这时CPU要调用调度程序，并且某个其他进程将运行。请求打印字符串的进程将被阻塞，直到整个字符串打印完。**

当打印机将字符打印完并且准备好接收下一个字符时，它将产生一个中断。这一中断将停止当前进程并且保存其状态。然后，打印机中断服务过程将运行，如果没有更多的字符要打印，中断程序将采取某个操作将用户进程解除阻塞。否则，它将输出下一个字符，应答中断，并且返回在中断之前正在运行的进程，该进程将从其停止的地方继续运行。

## 使用DMA的I/O

**中断驱动I/O的明显缺点是中断发生在每个字符上。中断要花费时间，所以这一方法将浪费一定数量的CPU时间。这一问题的一种解决方法是使用DMA。此时的思路是让DMA控制器一次给打印机一个字符，而不必打扰CPU。本质上，DMA是程序控制IO。只是由DMA控制器而不是主CPU做全部工作。这一策略需要特殊的硬件（DMA控制器），但是使CPU获得自由从而可以用在IO期间做其他工作。**

**DMA重大的成功是将中断的此时从打印每一个字符减少到打印每个缓冲区一次。如果有许多字符并且中断非常缓慢，那么采用DMA可能是重要的改进。另一方面，DMA控制器通常比主CPU慢很多。如果DMA控制器不能以全速驱动设备，或者CPU在等待DMA中断的同时没有其他事情要做，那么采用中断驱动IO甚至采用程序控制IO也许更好。**

