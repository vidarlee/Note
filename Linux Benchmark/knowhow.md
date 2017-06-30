- linpack是线性系统软件包(Linear system package)的缩写。现在在国际上已经成为最流行的用于测试高性能计算机系统浮点性能的benchmark。通过利用高性能计算机，用高斯消元法求解N元一次稠密线性代数方程组的测试，评价高性能计算机的浮点性能。

- NUMA,非统一内存访问(Non-uniform Memory Access),介于SMP(对称多处理)和MPP(大规模并行处理)之间，各个节点自有内存(甚至IO子系统),访问其它节点的内存则通过高速网络通道。NUMA信息主要通过BIOS中的ACPI(高级配置和编程接口)进行配置,Linux对NUMA系统的物理内存分布信息从系统firmware的ACPi表中获得，最重要的是SRAT(System Resource Affinity Table)和SLIT(System locality Information Table)表。SRAT表包含CPU信息、内存相关性信息,SLIT表则记录了各个节点之间的距离，在系统中由数组node_distance[]记录。这样系统可以就近分配内存，减少延迟。

- Sandy Bridge和Larrabee架构下的新指令集。
Intel的微架构进入了全速发展的时期，在2010年4月结束的IDF峰会上Intel公司就发布了2010年的RoadMap。2011年1月Intel发布了处理器微架构Sandy Bridge，其中全新增加的指令集也将带来CPU性能的提升。Intel公司将为Sandy Bridge带来全新的指令扩展集Intel Advanced Vector Extensions (Intel AVX)。AVX是在之前的128bit扩展到和256bit的SIMD(Single Instruction, Multiple Data)。而Sandy Bridge的SIMD演算单元扩展到256bits的同时数据传输也获得了提升，所以从理论上看CPU内核浮点运算性能提升到了2倍。
