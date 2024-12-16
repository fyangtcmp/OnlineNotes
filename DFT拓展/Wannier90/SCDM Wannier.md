*有些鸡肋，只对 spinless 系统效果较好，复杂的反铁磁系统很难拟合*

全自动wannier方法，可以在不需要指定投影子的情况下，进行全轨道wannier投影计算。该功能自vasp.6.3.0版本加入。先进行自洽计算，然后向INCAR中添加以下内容

```fortran
ICHARG= 11
ISYM= 0
LWANNIER90= T

LSCDM= T
CUTOFF_TYPE= gaussian # gaussian erfc ...
CUTOFF_SIGMA= 1.0
CUTOFF_MU= 2.2004

WANNIER90_WIN= "
num_iter=500
write_hr=true

bands_plot= .true.
bands_plot_format= gnuplot
begin kpoint_path
G  0            0           0       M  0.5          0.0         0.0
M  0.5          0           0       K  0.333333333  0.333333333 0.0
K  0.333333333  0.333333333 0.0     G  0            0           0
end kpoint_path
"
```

前3个关键词与传统方法一致，而新关键词LSCDM则启用SCDM方法计算，其他一些额外参数参见[LSCDM - VASP Wiki](https://www.vasp.at/wiki/index.php/LSCDM)

按该方法非自洽计算完成以后，会自动生成可以运行的 win 文件，可以直接启动 wannier90.x 进程。不过推荐在 win 文件中加入以下内容

```fortran
num_iter= 500
write_hr= .true.

bands_plot= .true.
bands_plot_format= gnuplot
begin kpoint_path
G  0            0           0       M  0.5          0.0         0.0
M  0.5          0           0       K  0.333333333  0.333333333 0.0
K  0.333333333  0.333333333 0.0     G  0            0           0     
end kpoint_path
```

此外，该方法生成的hr.dat是不包含任何对称性的