## VASP 5.4.4版

### 编译

若需对接最新版本（3.1.0）的 Wannier90，需要使用打[补丁]( https://github.com/Chengcheng-Xiao/VASP2WAN90_v2_fix "Chengcheng-Xiao/VASP2WAN90_v2_fix: An updated version of the VASP2WANNIER90v2 interface (github. Com)")后重新编译的版本
### 计算

#### 1 静态自洽

Wannier90 的拟合质量，以及 wannier90_hr.dat 文件的大小，都与 K 点密度呈正相关。因为 hr.dat 文件的大小直接决定了计算时长，因此在重度计算的情况下，必须在保证拟合质量的前提下，取尽量少的 k 点。**不过对于具有周期性的长轴，k 点数目最低不能少于 3**

#### 2 投影能带（可选）

当我们不知道投影区间的能带成分时，需要做一个 fat-band 能带计算，或者分波态密度计算，以选择 projectors 和能带窗口用来构造 Wannier 函数。熟练以后可以直接从能带数目的分布，结合 POTCAR 中的价电子成分，猜测出投影子。
#### 3 对接 Wannier90

如果投影能带发现区间内的能带总数少于投影子的数目，这是因为远离费米面处的导带数目是 VASP 随意取定的，可以手动指定NBANDS以增加能带总数。注意NBANDS必须可以被并行核数整除

编写wannier90.win文件。添加INCAR参数（关闭 NCORE），再运行 VASP

```fortran
NCORE  = 1
ICHARG = 11
ISYM   = -1
NBANDS = 168

LWANNIER90 = .TRUE.
```

顺利的话，会产生wannier90.amn, wannier90.mmn, wannier90.eig等3个文件。wannier90.win的内容也会被改变。

如果计算自旋霍尔电导，后处理需要 `wannier90.spn` 文件，需要串行运行 VASP，通过补丁提供的 tag ` LWRITE_SPN = .TRUE. ` 生成。因为只支持串行计算中写入，所以计算很慢。或者通过外部的 python 处理程序 [SelimLin/vasp2spn: Generating wannier90.spn from VASP file with PAW all-electron wavefunctions. (github.com)](https://github.com/SelimLin/vasp2spn)

ICHARG=11是为了加速收敛，某些磁性体系自洽收敛很困难，即使从上一步已经收敛的结果出发重新做自洽，也需要迭代很多步，因此采用非自洽。

ISYM需要关闭，原因参见 vasp 官网“Known issues”页面的说明
> **Interface to Wannier90 and PEAD calculations lead to incorrect results for non-collinear spin calculations when symmetries are used**: The rotation of the spinor part of the wavefunctions was missing which leads to incorrect results when computing the projections and overlaps written to the AMN and MMN files used by Wannier90 when LNONCOLLINEAR=.TRUE. and ISYM>=0 are set in the INCAR file. The fix for previous versions is to use ISYM=-1.

#### 4 运行wannier90主程序
```
mpirun -np 16 wannier90.x wannier90
```

## VASP 6.4.3 版

参见 VASP 官网“Known issues”页面，以及本人实际测试，VASP6 更新带来了很多对接 wannier 的问题，所以必须使用标题所述的最新版（截止 2024 年 9 月）
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

使用内部模式（library mode），直接得到 `hr.dat` 文件，不再需要手动执行 `wannier90.x`

```fortran
NCORE  = 1
ICHARG = 11
NBANDS = 168

LWANNIER90_RUN = .TRUE.
LWRITE_SPN = .TRUE.

NUM_WANN = 72

WANNIER90_WIN="
exclude_bands= 1-48

begin projections
W:d
Te:p
end projections

dis_win_min  = -4
dis_froz_min = -1
dis_froz_max =  1
dis_win_max  =  10

iprint=2
dis_num_iter=200
num_print_cycles=200
num_iter=0

write_hr=.true.
guiding_centres=.true.
write_xyz=.true.
translate_home_cell=.false.
use_ws_distance=.false.
"
```

与 VASP 5.4.4 的几点不同：
1. 使用 `LWANNIER90_RUN` 直接在 VASP 运行过程中进行 wannier 拟合
2. 官方支持了 `LWRITE_SPN` 功能，可以在并行计算中正常写入了
3. `NUM_WANN` 必须直接写入INCAR以便 VASP 识别
4. 以前需要预先写在 wannier90.win 中的信息，现在都放在 `WANNIER90_WIN` 这个 tag 中，由 VASP 自动创建 `wannier90.win` 并写入。注意 `num_bands` 和 `num_wann` 不再需要了，由 VASP 根据 `exclude_bands` 自动计算
5. 修正了上文中提到的需要关闭对称性的 bug（存疑）
