贝里曲率自洽SOC计算得到WAVECAR，可能需要ISYM=-1

使用vaspberry计算得到dat数据文件，ii、if指定计算能带的序号范围

kx、ky指定计算网格的密度，但是容易报错，建议不使用这两个参数

```
vaspberry -ii nmin -if nmax
vaspberry -ii nmin -if nmax -kx nx -ky ny
```

后处理程序绘图
```
python contour.py
```

圆二色谱
```
vaspberry -cd 1 -ii nmin -if nmax
```