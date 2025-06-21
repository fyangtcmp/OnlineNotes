## 一些细节
新版 wannier tools 使用原子单位制，长度单位 Bohr，能量单位 Hartree

按照 wannier tools 官网的说法，“选择轨道”不需要考虑自旋
>In order to get the weight onto the projections, we need to set SELECTED_WANNIERORBITALS card. it The orbital index is corresponding to the PROJECTOS card, nothing related to spin freedom. For example In Bi2Se3, there are 30 Wannier functions in the hr.dat, however, there are only 15 projectors (orbitals). So indices in SELECTED_WANNIERORBITALS are from 1 to 15.  

但是源码中按照 selected_atoms 翻译成轨道时，又变成了考虑自旋的，从 WT.out 就可以看出来

## ARPACK 安装
来自莱斯大学的原版 arpack 发布于 1996 年，曾经从互联网上短暂消失过一段时间，这是最近重新出现的一个源 https://www.arpack.org/home ，尚不清楚是否为原作者所有。编译需要仔细修改 ARmake.inc 中的内容

另外有一个由民间小组维护的更新至现代编译框架的 arpack 版本
[opencollab/arpack-ng: Collection of Fortran77 subroutines designed to solve large scale eigenvalue problems. (github.com)](https://github.com/opencollab/arpack-ng)
编译
```
sh bootstrap
./configure CC=icc CXX=icpc FC=ifort F77=ifort MPIFC=mpiifort MPIF77=mpiifort --enable-mpi --enable-static=yes --enable-shared=no
make
make check
sudo make install
```
默认情况下是只有动态库，但这样需要为每位用户配置库文件环境，所以改成静态库。如果同时编译静态和动态库，在链接 `.la` 文件时会报错

## 引入新功能

### main.f90

一系列if函数，判断输入文件的tag是否打开，调用对应的子程序
### module.f90

Fortran中所有变量必须预先定义，因此我们需要在module para中将新功能的tag作为logical变量定义在这里，然后加入下方的namelist / Control /
### readinput.f90 (可能不用加)

为所有CONTROL namelist的变量赋缺省值false
### \<subroutine\>.f90

实际用于实现的子程序，单独封装在一个f90文件中。需要注意，按照

[现代Fortran的推荐范式 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/100615040 "现代Fortran的推荐范式 - 知乎 (zhihu.com)")

的介绍，现代Fortran范式要求所有子程序封装在module中，且use必须使用only，以满足最小化原则，wannier tools这里因为历史遗留原因，很难遵循这一范式
### makefile

将\<subroutine>.f90加入makefile的list中，后于实现基础功能的模块，先于main.f90编译，以保证编译器正确处理依赖关系。