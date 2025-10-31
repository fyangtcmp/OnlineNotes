## 常用 conda 软件包
```
conda create -n py3 python=3.11 pymatgen spglib
pip install pyw90
```

## HDF5
设置环境变量
```bash
export CC=mpiicc
export CXX=icc
export FC=mpiifort
export CPP=cpp
export CFLAGS="-O3 -xHost"
export CXXFLAGS="-O3 -xHost"
export FCFLAGS="-O3 -xHost"
```
手动打开 Fortran 支持
```bash
./configure --prefix=/usr/local/ --enable-fortran
```
然后 make
在.bashrc 中加入以下内容
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```
## X11

[在Mac上使用远程X11应用 - 俺踏月色而来 - 博客园 (cnblogs.com)](https://www.cnblogs.com/andrewwang/p/8535081.html "在Mac上使用远程X11应用 - 俺踏月色而来 - 博客园 (cnblogs.com)")
## MATLAB

[Linux 命令行安装 matlab - scbox - 博客园 (cnblogs.com)]( https://www.cnblogs.com/scboy/articles/13206185.html "Linux 命令行安装 matlab - scbox - 博客园 (cnblogs.com)")

## FFTW3

```
./configure --prefix=/usr/local --enable-mpi --enable-openmp --enable-threads --enable-avx2 --enable-avx512 MPICC=mpiicc CC=icc F77=ifort
```

## spglib
需要关闭 test 的编译，因为它需要从 github 联网下载 googletest
```bash
cmake . -B ./build -DCMAKE_C_COMPILER=icc -DCMAKE_CXX_COMPILER=icpc -DCMAKE_Fortran_COMPILER=ifort -DSPGLIB_WITH_Fortran=true -DSPGLIB_WITH_TESTS=OFF
cmake --build ./build
sudo cmake --install ./build --prefix=<install_dir>
```
其中 fortran 模块 `spglib_f08.mod` 不会直接安装在 `/usr/local/include` 目录下，而是在一个子目录比如 `/usr/local/include/Spglib_Fortran/Intel-2021.5.0.20211109`，可以手动把它 cp 回上级目录
如果你使用的是 intel oneAPI 2025，也就是 llvm 编译器，有可能导致 fortran 模块不会正确安装，还要进入到目录里再手动编译安装一次
```bash
cd fortran
cmake . -B ./build 
cmake --build ./build
sudo cmake --install ./build
```
最终安装的 mod 文件位置也不太一样
## quspin
```bash
conda create -n quspin python=3.10 numpy=2.0.0 dill pip
conda activate quspin
pip install -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple quspin
```
如果出现环境包里没有 quspin-extension 的情况，可能是之前安装残留的包不在这个环境里，但 pip 也错误识别成存在了，需要手动 uninstall 掉 quspin 和 quspin-extension 然后再重新装