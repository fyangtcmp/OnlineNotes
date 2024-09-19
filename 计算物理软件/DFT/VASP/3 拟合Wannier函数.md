
## VASP 5.4.4版

### 编译

若需对接最新版本（3.1.0）的 Wannier90，需要使用打[补丁]( https://github.com/Chengcheng-Xiao/VASP2WAN90_v2_fix "Chengcheng-Xiao/VASP2WAN90_v2_fix: An updated version of the VASP2WANNIER90v2 interface (github. Com)")后重新编译的版本

### 计算

#### 1 静态自洽

Wannier90 的拟合质量，以及 wannier90_hr.dat 文件的大小，都与 K 点密度呈正相关。因为 hr.dat 文件的大小直接决定了计算时长，因此在重度计算的情况下，必须在保证拟合质量的前提下，取尽量少的 k 点。不过对于具有周期性的长轴，k 点数目最低不能少于 3

#### 2 投影能带（可选）

当我们不知道投影区间的能带成分时，需要做一个 fat-band 能带计算，或者分波态密度计算，以选择 projectors 和能带窗口用来构造 Wannier 函数。熟练以后可以直接从能带数目的分布，结合 POTCAR 中的价电子成分，猜测出投影子。

#### 3 对接 Wannier90

如果投影能带发现区间内的能带总数少于投影子的数目，这是因为远离费米面处的导带数目是vasp随意取定的，可以手动指定NBANDS以增加能带总数。

注意NBANDS必须可以被并行核数整除。

2. 编写wannier90.win文件。关闭NCORE

v3.0版本以后不再需要mp\_grid、spinors、原子坐标、晶格常数等参数，vasp会自动传给wannier，仅需要能量窗口、投影子等信息。如果使用qe等其他软件则依然要写入完整的信息。

能量窗口为绝对能量，而非相对于费米面的能量。

3. 添加INCAR参数（删除重复的参数），再运行VASP，NBANDS与OUTCAR中保持一致。

```fortran
ICHARG = 11
LWAVE  = .FALSE.
LCHARG = .FALSE.

ISYM = -1 # 0 
NBANDS=168

NCORE=1
LWANNIER90=.TRUE.
```

顺利的话，会产生wannier90.amn, wannier90.mmn, wannier90.eig等3个文件。wannier90.win的内容也会被改变。

如果计算自旋霍尔电导，后处理需要wannier90.spn文件，需要串行运行vasp，通过LWRITE\_SPN = .TRUE.生成，可配合NELM=0。因为只支持串行计算中写入，所以计算很慢

ICHARG=11是为了加速收敛，某些磁性体系自洽收敛很困难，即使从上一步已经收敛的结果出发重新做自洽，也需要迭代很多步，因此采用非自洽。

ISYM需要关闭，原因参见

[Phys. Rev. Materials 2, 103805 (2018) - Automated construction of symmetrized Wannier-like tight-binding models from ab initio calculations (aps.org)](https://journals.aps.org/prmaterials/abstract/10.1103/PhysRevMaterials.2.103805 "Phys. Rev. Materials 2, 103805 (2018) - Automated construction of symmetrized Wannier-like tight-binding models from ab initio calculations (aps.org)")

虽然 ISYM=0 或 -1 会增加布里渊区的 K 点数目，增大收敛难度，但与 ICHARG=11搭配使用可以部分解决这一问题。大多数情况下，ISYM=0 或 -1 能够在不改变 hr 文件大小的情况下显著提高 wannier 拟合质量。

4. 运行wannier90主程序

wannier90.x wannier90

5. 快速绘图

gnuplot -p wannier90\_band.gnu

  

## VASP 6.x.x 版

### 编译

1. 编译串行版本的wannier90-3.1.0库文件libwannier.a

2. 静态链接libwannier.a，即将makefile.include中以下内容

```bash
LLIBS += -L$(WANNIER90_ROOT)/lib -lwannier
```

修改为

```bash
LLIBS += $(WANNIER90_ROOT)/libwannier.a
```

### 计算

  

vasp 6.x 版本开始，对接wannier的逻辑有了较大改动。官方推荐通过LWANNIER90\_RUN=T，使用库模式 (library mode) 在vasp运行内部调用wannier90，直接输出hr.dat文件。但是基于同一组输入文件的测试结果显示，vasp 6.x内部wannier90解纠缠存在非常大的问题，wannier函数展宽远高于从外部手动调用wannier90.x的计算结果，即使在展宽貌似较小的情况下，绘制wannier能带也可能发现明显的错误。虽然LWANNIER90=T这一tag保留，但电子步结束后进入的依然是library mode，尚不清楚如何进入传统模式 (legacy mode) 。因此，目前已知最稳健的对接方式为：


```bash
ISTART=2
LWAVE= .FALSE.
LCHARG=.FALSE.

ISYM=0
ICHARG=11
NBANDS=168

LWANNIER90_RUN=T
NUM_WANN=72
LWRITE_MMN_AMN=T
WANNIER90_WIN="
exclude_bands =
dis_froz_min =
...
"
```

前两组参数与vasp 5.4.4一致，第三组参数中，NUM\_WANN即wannier90.win中的wannier轨道数，library mode下必须直接写入INCAR以便vasp识别；同时，library mode下为节省空间，mmn和amn信息默认不写入硬盘，所以这里打开LWRITE\_MMN\_AMN=T；最后，传统两步wannier对接法中保存在wannier90.win中的信息，都需要放在WANNIER90\_WIN=""中，由vasp自动创建wannier90.win并写入。

运行完成后，继续运行

```bash
wannnier90.x wannier90
```

重新得到hr.dat