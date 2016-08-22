##四种主要多核并行编程方法分析与比较 #

1、Message Passing Interface(MPI)

>suport:C/C++/Fortran

>优点：可以在集群上使用，也可以在单核/多核CPU上使用，它能协调多台主机间的并行计算，因此并行规模上的可伸缩性很强，能在从个人电脑到世界TOP10的超级计算机上使用。

>缺点：第一，基于消息传递，需要显示划分和分布计算任务，显示进行消息传递与同步，且不易增量开发串行程序的并行性；第二，使用进程间通信的方式协调并行计算，这导致并行效率较低、内存开销大、不直观、编程麻烦。


2、Open Multi Processing(OpenMP)

>suport:C/C++/Fortran

>优点：第一，共享存储模型，使得程序员不必进行数据划分和分布，使得开发并行程序比较容易；第二，更适合于SMP系统；第三，主要面向循环级的并行开发，可以容易地实现增量性的并行化。

>缺点：第一，OpenMP只适用于SMP结构；第二，OpenMP主要开发循环级的并行程序，受此限制，对某些应用并不适合；第三，OpenMP的编写、正确性调试和性能调度复杂。

3、Integrated Performance Primitives(Intel IPP)

Intel集成性能基元是Intel函数库的第二代。Intel为每种新的多核处理器都发布一个IPP函数库（C/C++ API），专用于多核架构，提供了调度优化的函数库，其中涉及的领域有数学、信号处理、音频视频、图像处理与编码、字符串、密码学。

>suport:C/C++

>　　优点：是经过性能高度优化的库，执行效率高。

>　　缺点：专用于Intel处理器和某些领域，不方便移植。


4、Threading Building Blocks(Intel TBB)

>suport:C/C++

>　　优点：可移植、可扩展。

>　　缺点：性能没有IPP高。

## Intel Threading Building Blocks(TBB)##

INTRODUCE：多核处理器的C++库，线程级的并行

>TBB 提供的接口：由底层到高层

>task_scheduler----->concurrent_container----->parallel_for----->pipeline 

1、并发容器（concurrent_container）

大部分程序都有容器类，在多线程环境下就有数据污染的问题，为了使并发的线程串行化，一般是使用加锁的办法，如果这个
容器由程序员自己来实现，难度还是比较大的，这样就需要有线程安全的容器类。

1、concurrent_hash_map

hash接口与stl类似

2、concurrent_vector

grow_by(n) 插入n个item（动态增长）

grow_to_at_least（）设定容器的大小

size()  包括正在并发增长的部分 因为有可能会同时取，所以程序员需要自己维护自己的class的线程安全性

clear() 不是线程安全的

3、concurrent_queue

pop_if_present(item) 非阻塞，

pop（） 阻塞，

concurrent_queue::size() 负数时表示有多少个消费者在等待

set_capacity（）指定队列大小，会使push操作被阻塞

在并行时，paralell_while pipeline 的效率要高于concurrent_queue 


2、parallel


| function         |parallel_for      |parallel_reduce    |parallel_while   |
| ----------------:| ----------------:| ----------------:| ----------------:|
| Introduction     | 多个数据或请求没有依赖关系，所要进行的操作是一样的|适合于需要汇总的情况，例各个数据的结果需要汇总回来|不知道循环何时结束，例使用for的end未知|

**Link(download)**:[https://www.threadingbuildingblocks.org/download#development-releases](https://www.threadingbuildingblocks.org/download#development-releases)

安装

- 下载：windows release版tbb
- 配置环境变量PATH，保证运行时找到dll
- 在TBB应用工程中添加tbb包含目录，即tbb相关头文件
- 在TBB应用工程中添加tbb库目录，即tbb的lib文件，注意32位和64位有分别的目录