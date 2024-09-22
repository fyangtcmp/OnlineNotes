## VASP 5.4.4版

### 编译

若需对接最新版本（3.1.0）的 Wannier90，需要使用打[补丁]( https://github.com/Chengcheng-Xiao/VASP2WAN90_v2_fix "Chengcheng-Xiao/VASP2WAN90_v2_fix: An updated version of the VASP2WANNIER90v2 interface (github. Com)")后重新编译的版本

### 计算

#### 1 静态自洽

Wannier90 的拟合质量，以及 wannier90_hr.dat 文件的大小，都与 K 点密度呈正相关。因为 hr.dat 文件的大小直接决定了计算时长，因此在重度计算的情况下，必须在保证拟合质量的前提下，取尽量少的 k 点。不过对于具有周期性的长轴，k 点数目最低不能少于 3

#### 2 投影能带（可选）

当我们不知道投影区间的能带成分时，需要做一个 fat-band 能带计算，或者分波态密度计算，以选择 projectors 和能带窗口用来构造 Wannier 函数。熟练以后可以直接从能带数目的分布，结合 POTCAR 中的价电子成分，猜测出投影子。
#### 3 对接 Wannier90

如果投影能带发现区间内的能带总数少于投影子的数目，这是因为远离费米面处的导带数目是vasp随意取定的，可以手动指定NBANDS以增加能带总数。注意NBANDS必须可以被并行核数整除

编写wannier90.win文件。添加INCAR参数（关闭 NCORE），再运行VASP，NBANDS与OUTCAR中保持一致。

```fortran
ICHARG = 11
LWAVE  = .FALSE.
LCHARG = .FALSE.

ISYM = -1
NBANDS=168

NCORE=1
LWANNIER90=.TRUE.
```

顺利的话，会产生wannier90.amn, wannier90.mmn, wannier90.eig等3个文件。wannier90.win的内容也会被改变。

如果计算自旋霍尔电导，后处理需要wannier90.spn文件，需要串行运行vasp，通过LWRITE\_SPN = .TRUE.生成，可配合NELM=0。因为只支持串行计算中写入，所以计算很慢

ICHARG=11是为了加速收敛，某些磁性体系自洽收敛很困难，即使从上一步已经收敛的结果出发重新做自洽，也需要迭代很多步，因此采用非自洽。

ISYM需要关闭，原因参见 vasp 官网“Known issues”页面的说明
> **Interface to Wannier90 and PEAD calculations lead to incorrect results for non-collinear spin calculations when symmetries are used**: The rotation of the spinor part of the wavefunctions was missing which leads to incorrect results when computing the projections and overlaps written to the AMN and MMN files used by Wannier90 when LNONCOLLINEAR=.TRUE. and ISYM>=0 are set in the INCAR file. The fix for previous versions is to use ISYM=-1.

#### 4 运行wannier90主程序
```
mpirun -np 16 wannier90.x wannier90
```

## VASP 6.4.3 版

同样参见 VASP 官网“Known issues”页面，VASP6 更新带来了很多对接 wannier 的问题，所以必须使用最新版
### 编译

按照 VASP 官网的说明，编译串行版本的 wannier90 库文件 `libwannier.a`。在 `makefile.include` 中写入 wannier90 的安装位置，并将以下内容
```bash
LLIBS += -L$(WANNIER90_ROOT)/lib -lwannier
```
修改为
```bash
LLIBS += $(WANNIER90_ROOT)/libwannier.a
```

### 计算

VASP 6 版本开始，对接wannier的逻辑有了较大改动。官方推荐通过 `LWANNIER90_RUN=T`，使用库模式 (library mode) 在 VASP 运行内部调用 wannier90，直接输出 hr.dat 文件。但是基于同一组输入文件的测试结果显示，VASP 6 内部 wannier90 解纠缠存在非常大的问题，wannier 函数展宽远高于从外部手动调用 wannier90.x 的计算结果，即使在展宽貌似较小的情况下，绘制 wannier 能带也发现了明显的错误。

虽然LWANNIER90=T这一tag保留，但电子步结束后进入的依然是library mode，尚不清楚如何进入传统模式 (legacy mode) 。因此，目前已知最稳健的对接方式为：

```bash
ISTART=2
LWAVE= .FALSE.
LCHARG=.FALSE.

LWRITE_SPN = .TRUE.

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