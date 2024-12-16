## HDF5
最后没安成功...
```bash
./configure CC=icc CXX=icpc --prefix=/usr/local/hdf5 --enable-cxx=yes --enable-fortran=yes
```

在.bashrc 中加入以下内容
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/hdf5/lib
export PATH=$PATH:/usr/local/hdf5/bin
export CPATH=:$CPATH:/usr/local/hdf5/include
```
## X11

[在Mac上使用远程X11应用 - 俺踏月色而来 - 博客园 (cnblogs.com)](https://www.cnblogs.com/andrewwang/p/8535081.html "在Mac上使用远程X11应用 - 俺踏月色而来 - 博客园 (cnblogs.com)")
## MATLAB

[Linux 命令行安装 matlab - scbox - 博客园 (cnblogs.com)]( https://www.cnblogs.com/scboy/articles/13206185.html "Linux 命令行安装 matlab - scbox - 博客园 (cnblogs.com)")
