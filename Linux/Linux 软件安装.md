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
