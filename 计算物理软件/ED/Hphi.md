官网：[Quantum Lattice Model Simulator Package: HΦ | HΦ (u-tokyo.ac.jp)](https://www.pasums.issp.u-tokyo.ac.jp/hphi/en/)
编译：基于 cmake，需要注意 cmake 版本和 intel 编译器版本的兼容性，以下是一些测试结果
```
intel parallel studio 2019 (icc, ifort)
    cmake 2.8.12.2: failed
	cmake 3.14.3: passed
	
intel OneAPI 2024.1 (icx, ifx)
	cmake 3.22.1: failed
	cmake 
```