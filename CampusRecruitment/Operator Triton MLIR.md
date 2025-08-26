# 1.Operator
## 1.在算子组参与两代自研 AI 芯片(S60/1600)的算子研发工作
SIMD架构：full chip为单die，包含4个cluster/sic。每个cluster中包含1个cdte和7个sip。7个sip中的6个提供给TopsAten算子编程使用，1个提供给eccl用于做reduce，算子不可用
每个sip中有2个Kernel Slot，可各自发射一个**硬件线程**，有10个thread slot，可最多运行10个**软件线程**，这些都共享sip上的计算资源
在编程模型中，称硬件线程为thread，软件线程为subthread
==4block * 6SIP * 2kernel_slot * 4subThread==
## 2.析构问题导致的内存泄漏
(1) 当前方案的内存管理：如果数据非连续，需要CreateContiguousTensor，则把topsopTensorHandle_t保存。直到最后析构的时候调用topsopDestroyTensor(item, stream_)释放内存。
(2) 近期优化代码，把op_cumprod变成static变量。导致它的生命周期持续到线程结束。带来收益：连续调用N次topsatenCumprod接口，只会被初始化一次，减少开销。
(3) 影响：连续调用多次非连续的topsatenCumprod，内存会一直增加，但是由于cumprod是static，要等到线程结束才会释放。调用topsStreamDestroy(stream)后，析构函数释放内存，但是析构函数需要stream。
(4) 执行顺序：
	(a) topsStreamCreate(&stream)
	(b) topsatenCumprod(xxx)
	(c) topsStreamDestroy(stream)
	(d) 析构，释放内存
但是由于析构函数中的stream在调用算子api结束时已经被destory了，所以释放失败；

解决策略：借用现在的基类，起一个临时对象随着每个OP API调用结束而结束，去做实时的内存管理，这样调用完一次topsatenCumprod算子API后，就会走析构去释放op_cumprod Tensor，而不是等到整个线程执行结束
结果：经过测试，内存泄漏问题解决且未造成host端额外开销；
## 3.随机数算子实现与CPU模拟对照
一些小优化：Philox算法的优化，switch -> 数组(使用state & 3控制偏移)
添加philoxState.intragraph，offset_intragraph：每次调用会将philox内部的state自增1，四次调用(自增)相当于offset+1，也就是说(0, 0, 0)调用四次，在第五次调用时的结果与(0, 0, 1)第一次调用相同
相同的Seed + 相同的Offset → 相同的State → 相同的随机数序列
## 4.Reduce 类实现与优化（mean）
1.分块计算 (Tiling)
将输入 Tensor 的大维度分割成更小的数据块（Tiles）
每个 Block 线程负责将一个小数据块从​​全局内存​​加载到​​共享内存​​
在共享内存中进行局部归约（求和），将结果写回全局内存
2.双缓冲
隐藏全局内存访问的延迟。在计算当前数据块的同时，预加载下一个数据块到L1
3.计算流并行与 Unroll​
**Unroll 16次​**​: 在加载和归约循环中，手动展开循环（`#pragma unroll 16`）。这减少了循环开销，增加了指令级并行，让编译器能更好地调度指令，隐藏延迟
## 5.实现任意Dimensions上的Padding，包括constant、reflect 和 edge模式
Padding 的本质：​创建一个新的、更大的内存块，并将原始数据拷贝到其中的特定位置，然后用特定的值填充周围的空间
通过预先计算数据的分块（Slice）和拼接（Deslice）逻辑，将非连续的内存访问转化为可控的、可预测的局部操作


# 2.[Triton](https://github.com/tfruan2000/tfruan2000.github.io/blob/main/_posts/Triton/2024-04-11-triton-survey.md)
1. Triton 是Openai研发的专门为深度学习和高性能计算任务设计的编程语言和编译器，旨在简化并优化在GPU上开发高性能的深度学习算子的难度，提供比CUDA 更高生产力编写快速代码。 （编写灵活的DSL，降低GPU编程的难度同时提升算子效率） Triton的核心理念是基于分块的编程范式可以促进神经网络的高性能计算核心的构建，CUDA在线程的细粒度上进行编程，Triton是在分块（block/tile)的细粒度上进行编程
   比起CUDA的SIMT编程范式，由多个Thread并行处理，triton是SIMD编程范式，（基于block算法编程范式），省去线程之间的同步操作等
2. Triton 采用 [just-in-time](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Just-in-time_compilation) 机制进行编译。如果一个函数被 triton.jit 装饰器装饰, 那么在运行时会先编译成机器码, 然后再执行。我们将被 triton.jit 装饰的函数称为核函数。
   将核函数解析成AST ([抽象语法树](https://zhida.zhihu.com/search?q=%E6%8A%BD%E8%B1%A1%E8%AF%AD%E6%B3%95%E6%A0%91&zhida_source=entity&is_preview=1)) 
   根据 AST 生成 Triton IR 代码 
   将 Triton IR 代码转换成 Triton GPU IR 代码 
   将 Triton GPU IR 代码转换成 LLVM IR 代码 
   使用 [LLVM](https://link.zhihu.com/?target=https%3A//github.com/llvm/llvm-project), 将 LLVM IR 代码转换成 PTX 代码
   使用 [ptxas](https://link.zhihu.com/?target=https%3A//docs.nvidia.com/cuda/parallel-thread-execution/index.html), 将 PTX 代码转换成 cubin 机器码
   ```
   Triton IR → Triton GPU IR (MLIR Dialect) → LLVM IR (MLIR) → PTX
   高阶优化​​（如循环分块）在 Triton GPU IR 层完成。
   硬件相关优化​​（如寄存器分配）在 LLVM IR 层实现。
   ```
   

# 3.[MLIR](https://github.com/tfruan2000/tfruan2000.github.io/blob/main/_posts/MLIR/2023-06-22-mlir-survey.md)
## 3.1 [[MLIR Dialect]]理解
 - MLIR 的目标是 **可扩展**，任何框架或硬件都能定义自己的算子集。
- 如果 MLIR 内部只固定一套 IR，那扩展性会差；通过 **Dialect 机制**，不同领域（算术、张量代数、控制流、硬件指令集）都能共存于 MLIR。
- 不同 Dialect 可以在同一个 IR 中 **混合使用**，并且通过 **转换 Pass** 逐步“降级/优化”成底层 Dialect（比如 LLVM Dialect）
- 常见的dialect：builtin、bufferization、tensor、linarg、llvm、gpu、vector、math dialect
# 4.[Dataflow架构](https://mp.weixin.qq.com/s/q0Q15nbwbDU_8BGbH5l5LA)

