- linpack是线性系统软件包(Linear system package)的缩写。现在在国际上已经成为最流行的用于测试高性能计算机系统浮点性能的benchmark。通过利用高性能计算机，用高斯消元法求解N元一次稠密线性代数方程组的测试，评价高性能计算机的浮点性能。

- NUMA,非统一内存访问(Non-uniform Memory Access),介于SMP(对称多处理)和MPP(大规模并行处理)之间，各个节点自有内存(甚至IO子系统),访问其它节点的内存则通过高速网络通道。NUMA信息主要通过BIOS中的ACPI(高级配置和编程接口)进行配置,Linux对NUMA系统的物理内存分布信息从系统firmware的ACPi表中获得，最重要的是SRAT(System Resource Affinity Table)和SLIT(System locality Information Table)表。SRAT表包含CPU信息、内存相关性信息,SLIT表则记录了各个节点之间的距离，在系统中由数组node_distance[]记录。这样系统可以就近分配内存，减少延迟。

- Sandy Bridge和Larrabee架构下的新指令集。
Intel的微架构进入了全速发展的时期，在2010年4月结束的IDF峰会上Intel公司就发布了2010年的RoadMap。2011年1月Intel发布了处理器微架构Sandy Bridge，其中全新增加的指令集也将带来CPU性能的提升。Intel公司将为Sandy Bridge带来全新的指令扩展集Intel Advanced Vector Extensions (Intel AVX)。AVX是在之前的128bit扩展到和256bit的SIMD(Single Instruction, Multiple Data)。而Sandy Bridge的SIMD演算单元扩展到256bits的同时数据传输也获得了提升，所以从理论上看CPU内核浮点运算性能提升到了2倍。

### 硬链接
  不允许用户给目录创建硬链接。
  只有在同一个文件系统中的文件之间才能创建链接。

### 内核例程
  创建，撤销及同步现有进程的任务都委托给内核中的一组例程来完成。

### 内核线程
  Unix系统还包括几个所谓内核线程（kernel thread）的特殊进程（被赋予特殊权限的进程），它们具有以下特点：
- 它们以内核态运行在内核地址空间
- 它们不与用户直接交互，因此不需要终端设备
- 它们通常在系统启动时创建，然后一直处于活跃状态直到系统关闭。

### 进程实现
当内核暂停某个进程的执行时，就把几个相关处理器寄存器的内容保存在进程描述符中。这些寄存器包括：
- 程序计数器（PC）和栈指针(SP)寄存器
- 通用寄存器
- 浮点寄存器
- 包含CPU状态信息的处理器控制寄存器（处理器状态字，Processor Status Word）
- 用来跟踪进程对RAM访问的内存管理寄存器
