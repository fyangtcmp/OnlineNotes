## 简介
电子散射率（scattering rate），或者说弛豫时间（relaxation time）的倒数 $\tau^{-1}$ ，是决定材料许多非本征输运性质的非常重要的一个参数。然而，想要从计算上得到这一参数是十分困难的，一方面，它会受到具体的材料样品中杂质、缺陷等散射源的分布的影响，所以从某种程度上来说，电子散射率就不是一个第一性的参数；另一方面，第一性原理的计算电子散射需要考虑到电子-声子耦合效应（electron-phonon coupling），非常耗时。
在对精度要求不高的情况下，我们可以把弛豫时间视为一个常数，这就是最简单的常数弛豫时间近似（constant relaxation time approximation），它可以通过计算材料不含弛豫时间的纵向电导率，同实验上实际测定的纵向电导率相比较来得到。
在动量弛豫时间近似下（momentum relaxation time approximation），[AMSET](https://hackingmaterials.lbl.gov/amset/) 软件通过对公式的重构，提供了一种可以绕过电声耦合计算，从而大幅减少计算耗时的方法，可以高效计算材料中各种不同来源的电子散射率。
## 流程
### 1 结构优化
VASP 高精度结构优化 POSCAR，参见 [[2 结构稳定性分析]]
### 2 电子性质计算
在密格点上进行静态自洽计算，需要为常规 K 点密度的两倍以上，生成 `vasprun.xml` 和 WAVECAR 两个文件。
运行 `amset wave` 指令将 WAVECAR 中的波函数系数收集至 `wavefunction.h5` 文件内。这里默认的能量截断是 1.5eV，log 中会显示能量截断对应的能带指标范围，例如
```
Including bands 87—116
```
这个值需要记下来。
创建一个 `amset_res` 文件夹，用于最终的散射率计算，将刚才密格点静态自洽计算得到的 `vasprun.xml` 和 `wavefunction.h5` 放入其中
### 3 物理量计算
根据所要计算的散射率的类型，查阅 https://hackingmaterials.lbl.gov/amset/scattering/#summary-of-scattering-rates ，确定该性质所需要的其他输入量，以及在 AMSET 本体输入文件中的控制 tag (ADP，PIE 等等)
#### 3.1 声学形变势散射
声学形变势散射 (Acoustic deformation potential scattering) 需要掺杂下形变势的信息，以及弹性常数。流程非常类似于 phonopy 计算声子谱，准备好优化后的 POSCAR，用于静态自洽的 KPOINTS，POTCAR，INCAR。在 INCAR 中加入 `ICORELEVEL = 1` ，然后运行以下脚本
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
这种时候，就需要手动指定能带范围，重新生成 `deformation.h5` 文件。将该文件放入 `amset_res` 文件夹之中，也就是和上一步得到的 `vasprun.xml` 和 `wavefunction.h5` 放在一起。
```bash
amset deform read undeformed def-* -b 87:116
```
弹性常数计算，在静态自洽的 INCAR 中加入以下参数
```
! ENCUT= 400
NCORE = 1
EDIFF = 1E-8
PREC = Accurate
NSW = 1
IBRION = 6
ISIF = 3
```
具体参数设置的影响： `NCORE` 等并行参数极有可能会在**运行一段时间后**导致报错。`EDIFF` 从 1e-7 降至 1e-6 的影响在 1kBar 左右。`NFREE` 从 2 升至 4，计算耗时加倍，影响不超过 10kBar。SOC 的影响最明显，约在几十个 kBar，所以是不可忽略的。
这一步耗时可能会很久。结果记录在 OUTCAR 之中，搜索 `TOTAL ELASTIC MODULI` 即可找到，之后需要手动填入 AMSET 本体的输入文件之中。注意 VASP 输出的单位是 kBar，而 AMSET 采用的单位是 GPa，所以需要将 OUTCAR 中弹性常数的数值除以 10 。更简单的方法是直接用 vaspkit 203 功能提取出来，会自动转换为 GPa 单位。
#### 3.2 压电散射
压电散射 (Piezoelectric scattering) 需要**高频**介电常数和压电常数。在 INCAR 中加入以下参数
```
NCORE = 1
EDIFF = 1E-8
PREC = Accurate
NSW = 1
IBRION = 8
LEPSILON = True
```
这一步耗时可能会很久。结果记录在 OUTCAR 之中。
但是，查找 OUTCAR 之后，我们会发现结果中不存在高频介电常数，VASP wiki 对 `LEPSILON` 功能的解释也只是提及了静态介电常数。事实上，VASP 计算频率依赖的介电常数的功能是通过打开 `LOPTICS` 实现的，amset 开发者在 issue 的回答中也确认了这一点。
压电常数以 $C/m^2$ 为单位，分为电子贡献和离子贡献两部分，这里 VASP 没有给出求和后的结果，需要手动求和，也可以使用 vaspkit 442 功能提取，但目前版本 (vaspkit 1.5.1) 有一个 bug，它无法读取 VASP 5.4.4 输出的 OUTCAR，只能处理 VASP6 (已验证 6.3.1 和 6.5.1) 输出的 OUTCAR。==VASP wiki 记载 VASP 5.4.4 及之前的版本计算压电常数时存在符号错误==
#### 3.3 杂质散射
杂质散射 (Ionized impurity scattering) 需要**静态**介电常数，参数设置和压电散射计算一样，在 OUTCAR 中搜索 `MACROSCOPIC STATIC DIELECTRIC TENSOR` 即可找到。也可以使用 vaspkit 442 提取。
完整输入参数列表显示杂质散射计算还会涉及 `defect_charge` 和 `compensation_factor` 这两个参数，不过在官方 example 里没有使用，这里也就不做讨论，直接使用默认值。
但目前测试显示，即使较小的散射浓度，也会导致计算出现 NaN 非法值，进而导致后续电导率计算失败，程序崩溃。
### 4 散射率计算
现在就可以正式开始散射率计算了。AMSET 有 python 脚本和命令行两种调用方式，这里以命令行方式为例，我们需要创建 `settings.yaml` 参数文件 (常数的数值仅为示意，具体非零元素的位置由晶体对称性决定)
```yaml
# general settings
scattering_type: [ADP, PIE]
doping: [-2e22, -8e20, 0]
temperatures: [30]
bandgap: 0

static_dielectric: 
  - [144,  53,  53]
  - [ 53, 144,  53]
  - [ 53,  53, 144]

high_frequency_dielectric: 
  - [144,  53,  53]
  - [ 53, 144,  53]
  - [ 53,  53, 144]
 
piezoelectric_constant:
- [144,  53,  53, 0, 0, 0]
- [ 53, 144,  53, 0, 0, 0]
- [ 53,  53, 144, 0, 0, 0]
  
# electronic_structure settings
interpolation_factor: 10

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
`nworkers` 指定散射率计算的并行核数，但最开始的初始化、计算 DOS、散射率预处理等步骤依然会是单核串行的。此外官方建议执行 `export OMP_NUM_THREADS=1` 以预先关闭 python 中 OPENMP 的全局线程并行。
`interpolation_factor` k 点插值数量，这是控制计算精度的首要因素。默认值为 10，官方 example 中设置的值为 50，根据有限的测试结果，提高插值数量影响在 10% 以内。但要注意，对于包含 SOC 的大体系，做过密的插值可能会吃掉巨量的内存！因此建议从一个较小的值出发，时刻监控程序运行的情况，等到正式开始并行环节后内存占用量才会保持稳定。或者直接放弃在 AMSET 计算中考虑 SOC 效应，根据有限的测试结果，影响在 30% 以内。
执行
```bash
amset run
```
以进行计算
### 5 散射率合成
拓展的马西森定律 (Matthiessen’s rule) ：金属中不同来源的散射贡献是相互独立的，总散射截面是不同来源散射的算术叠加，而弛豫时间是散射率的倒数，因此总弛豫时间可以表达为
$$
\frac{1}{\tau_{total}}=\sum_i\frac{1}{\tau_i}
$$
**缺乏可靠的文献出处，原始的马西森定律是一个经验定律，且只涉及声子散射和杂质散射两种机制**
