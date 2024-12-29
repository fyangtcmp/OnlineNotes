## 简介
电子散射率（scattering rate），或者说弛豫时间（relaxation time）的倒数 $\tau^{-1}$ ，是决定材料许多非本征输运性质的非常重要的一个参数。然而，想要从计算上得到这一参数是十分困难的，一方面，它会受到具体的材料样品中杂质、缺陷等散射源的分布的影响，所以从某种程度上来说，电子散射率就不是一个第一性的参数；另一方面，第一性原理的计算电子散射需要考虑到电子-声子耦合效应（electron-phonon coupling），非常耗时。
在对精度要求不高的情况下，我们可以把弛豫时间视为一个常数，这就是最简单的常数弛豫时间近似（constant relaxation time approximation），它可以通过计算材料不含弛豫时间的纵向电导率，同实验上实际测定的纵向电导率相比较来得到。
在动量弛豫时间近似下（momentum relaxation time approximation），AMSET 软件通过对公式的重构，提供了一种可以绕过电声耦合计算，从而大幅减少计算耗时的方法，可以高效计算材料中各种不同来源的电子散射率。
## 流程
### 1 结构优化
VASP 高精度结构优化 POSCAR，参见 [[2 结构稳定性分析]]
### 2 电子性质计算
在密格点上进行静态自洽计算，需要为常规 K 点密度的两倍以上，生成 `vasprun.xml` 和 WAVECAR 两个文件。
运行 `amset wave` 指令将 WAVECAR 中的波函数系数收集至 `wavefunction.h5` 文件内。这里默认的能量截断是 1.5eV，log 中会显示能量截断对应的能带指标范围，例如
```
Including bands 87—116
```
这个值需要记下来
### 3 力学性质计算
根据所要计算的散射率的类型，查阅 https://hackingmaterials.lbl.gov/amset/scattering/#summary-of-scattering-rates ，确定程序所需要的力学输入量，这里以声学形变势散射为例，它需要掺杂下形变势的信息，以及弹性常数

形变势计算的流程非常类似于 phonopy 计算声子谱，准备好优化后的 POSCAR，用于静态自洽的 KPOINTS，POTCAR，INCAR。在 INCAR 中加入 `ICORELEVEL = 1` ，然后运行以下脚本
```bash
np=9
amset deform create

mkdir undeformed
cp POSCAR INCAR KPOINTS POTCAR undeformed
cd undeformed || exit
mpirun -np $np vasp_ncl > log.vasp.out
cd ..

for i in POSCAR-*
do
    workdir=def-${i#*-}
    mkdir "$workdir"
    cp "$i" "$workdir"/POSCAR
    cp INCAR KPOINTS POTCAR "$workdir"
    cd "$workdir" || exit
    mpirun -np $np vasp_ncl > log.vasp.out
    cd ..
done

amset deform read undeformed def-*
```
脚本将自动对原始的和位移后的 POSCAR 进行静态自洽计算，并导出形变势信息至 `deformation.h5` 文件中。
这里默认的能量截断也是 1.5eV，但因为之前电子性质计算使用的是密格点，而这里形变势计算使用的是正常格点，在能带结构较为复杂的情况下，相同能量截断，在不同 K 点网格上对应的能带指标范围可能会有所偏差。检查这一步的 log，可能会出现
```
Including bands 87—114
```
这种时候，就需要手动指定能带范围，重新生成 `deformation.h5` 文件
```bash
amset deform read undeformed def-* -b 87:116
```

弹性常数计算，在静态自洽的 INCAR 中加入以下参数
```
PREC = Accurate
ENCUT= 400
NSW = 1
IBRION = 6
ISIF = 3
```
其他一些参数的影响： `NCORE` 等并行参数可能会导致报错。`EDIFF` 从 1e-6 降至 1e-7 的影响在 1kBar 左右。`NFREE` 从 2 升至 4，计算耗时加倍，影响不超过 10kBar。SOC 的影响比较明显，在几十个 kBar 左右，所以是不可忽略的
最终结果记录在 OUTCAR 之中，搜索 `SYMMETRIZED ELASTIC MODULI` 即可找到

### 4 散射率计算
将密格点静态自洽计算得到的 `vasprun.xml` 和 `wavefunction.h5` ，记录形变势信息的 `deformation.h5` ，一共三个文件放在同一个文件夹下，现在就可以正式开始 AMSET 计算了。AMSET 有 python 脚本和命令行两种调用方式，这里以命令行方式为例，我们需要创建 `settings.yaml` 参数文件
```yaml
# general settings
scattering_type: [ADP]
doping: [-1e18, 0, 1e18]
temperatures: [5]
bandgap: 0

# electronic_structure settings
interpolation_factor: 50

# materials properties
deformation_potential: deformation.h5
elastic_constant:
  - [144,  53,  53,  0,  0,  0]
  - [ 53, 144,  53,  0,  0,  0]
  - [ 53,  53, 144,  0,  0,  0]
  - [  0,   0,   0, 75,  0,  0]
  - [  0,   0,   0,  0, 75,  0]
  - [  0,   0,   0,  0,  0, 75]

# performance settings
nworkers: 16

# output settings
write_mesh: False
file_format: txt
```
注意这里弹性常数的单位是 GPa，而 VASP 输出的单位是 kBar，所以需要将 OUTCAR 中弹性常数的数值除以 10 后填在这里
`nworkers` 指定散射率计算的并行核数，不要忘记使用 `export OMP_NUM_THREADS=1` 关闭 python 中 OPENMP 的全局线程并行
