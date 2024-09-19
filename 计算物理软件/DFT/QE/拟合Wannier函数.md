## 自洽计算

自动撒点，自洽计算

## 非自洽计算

使用 kmesh.pl 脚本生成的高密度固定的 K 点网格，非自洽计算。在 wannier90的输入文件.win 文件中也需要写入相同的 k 点网格，注意格式差别。

## Wannier90处理

为格式统一，以下 wannier90 的\<seedname\>固定为'wannier90'
wannier90处理得到接口文件，这一步速度很快，可以直接串行，超算可直接在主节点运行

```bash
wannier90.x -pp wannier90
```

pw2wannier90将qe波函数转化为wannier90可读取的格式
编辑输入文件 pw2wan.in

```
&inputpp
  outdir     =  './'
  prefix     =  '<prefix>'   
  seedname   =  'wannier90'   
  write_amn  =  .true.
  write_mmn  =  .true.
/
```

然后运行

```bash
 mpirun -np 16 pw2wannier90.x -in  pw2wan.in  >  pw2wan.out
```

正式运行 wannier90

```bash
 mpirun -np 16 wannier90.x wannier90
```

运行 postw90 根据 wannier 函数计算具体物理量