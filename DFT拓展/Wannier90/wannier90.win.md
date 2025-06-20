wannier90.win 是 Wannier90 软件控制参数的输入文件，地位类似于 VASP 的 INCAR 文件。它的正式名称为 `<seedname>.win`，在对接 QE、Wien2k、OpenMX 等输入文件名可变的 DFT 软件时，它的前缀也是可变的。但 VASP 所有输入输出文件的名称都是固定的，所以 `<seedname>.win` 也不能免俗，被固定为  `wannier90.win`
我们先给出一个常规输入文件的范例

```
num_wann     = 88
num_bands    = 88
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
```

`num_wann` 所有投影子的总数，它等于每个原子的投影轨道数 $\times$ 原胞中原子个数 $\times$ (若有SOC，2倍)，求和。具体轨道的选取需要从 DFT 计算结果做投影能带分析

`num_bands` 大于等于 `num_wann`

`exclude_bands` 不需要的能带序号范围，一般来说可以排除掉所有的价带深能级。`num_bands + exclude_bands` 的数目，要等于 INCAR 中 `NBANDS` 的数目

`projections` 中的元素顺序请按照 POSCAR 中的元素顺序来写。新版 wannier90 默认是按照 `projections` 的定义，按照**元素-原子-轨道-自旋**的层级来排列 hr.dat 中的轨道顺序的，但有些后处理程序没有正确的处理这一点，反而按照 POSCAR 的顺序来排列轨道。因此保持两种文件的元素顺序一致可以防止触发低级错误

`dis_froz_min` `dis_froz_max` 不能选的太小，虽然减小窗口容易使得总展宽的数值减小，但有可能使窗口外的区域拟合程度下降，所以必须包含我们所希望拟合的全部能带范围。能量窗口为绝对能量，而非相对于费米面的能量。如果 `num_wan` = `num_band`，wannier90 不解纠缠，能量窗口参数将不起作用。

`iprint` 输出信息的详细程度，1 为默认值；2 会提供所有投影子的坐标、量子数、局域坐标系等信息，在定义 `projections` 时使用简写的情况下特别有用，可以清楚的了解到自旋和轨道究竟是按什么顺序排列的（早期版本的 wannier90 优先排列同自旋的轨道，新版 wannier90 则优先排列同轨道的两个自旋上下的投影子）；3 提供的信息更加细节，一般只在 debug 时使用

`dis_num_iter` 解纠缠轮数，不是越大越好，默认的200轮在大多数情况下都是一个较优的选择。过高的轮数，比如600轮，虽然表面上降低了 wannier 函数的展宽，但很容易发生过拟合，将重要的一些定性性质，比如能带的二重简并性破坏
  
`num_iter` 最大局域化轮数，可以明显增强 wannier 拟合的效果，但会使得 wannier center 的位置偏离原子坐标，破坏对称性。所以如果需要后续对称化 wannier 的情况下必须设为 0。顺便一提，虽然 wannier90 内部提供两种宣称能够保持对称性的 wannier 函数构造方法，其一是symmetry-adapted Wannier functions，其二是 selectively localized Wannier functions，但前者只适用于 spinless 的系统，后者无法固定所有的投影子，必须保留一定的自由度，所以均不适合复杂磁性系统

`guiding_centres` 帮助减少展宽，参见 [wannier90拟合之guiding_centre对spread的影响 (qq.com)](https://mp.weixin.qq.com/s/qRRNWmGjgYcZgGECdgbJdg)

`mp_grid`、`spinors`、原子坐标、晶格常数等参数，VASP 会自动传给 wannier90，如果使用 qe 等其他软件，则依然要手动写入完整的信息。
 
`use_ws_distance=.false.` 如果使用 use_ws_distance，就需要使用 wsvec 去修正跃迁矢量。所以如果不能处理 wsvec 的情况下，应该关掉 use_ws_distance