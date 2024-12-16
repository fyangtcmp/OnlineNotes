目前实现 GPU Fortran 计算主要有三种方法：
1. 基于源生 Fortran2023 规范的英伟达编译器自动并行化
2. 基于 OpenACC 导语的编译器并行化
3. 基于 CUDA Fortran ，完全由用户显式控制 CPU 和 GPU 上的内存管理的细粒度并行
对于 wannier 计算这种需要对 k 点并行, 每个 k 点上矩阵维度只有 $10^2$ 量级的情况，GPU 优化效率并不高，而且 wannier 计算需要反复调用 blas 线性代数库与矩阵本征值库，因此 k 点本身无法在 GPU 上并行