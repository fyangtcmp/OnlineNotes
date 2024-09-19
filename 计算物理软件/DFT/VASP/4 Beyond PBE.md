PBE 泛函，是目前固体物理研究领域大多数 DFT 计算默认采取的泛函，它的性能最为均衡，但也有某些性质是它描述的不太好的，一个典型的例子就是带隙。 PBE 泛函倾向于低估带隙，对于磁性的过渡金属化合物而言，这种带隙的低估常常是由金属的关联效应引起的，可以尝试 [[0 输入文件#DFT+U|DFT+U]]。对于非磁化合物而言，可能就需要考虑使用 meta-GGA 泛函，如 mBJ，或者杂化泛函，如 HSE06，或者更进一步使用 GW 方法修正电子的关联效应。
需要注意这些 beyond PBE 的方法都是非常耗时的。以 HSE06 为例，综合网络上流传的经验和本人测试，它的计算耗时约为 PBE 的几百~上千倍左右，内存消耗在相同 k 点的情况下约为 4 倍左右。因此推荐从简单的 mBJ 泛函开始尝试，然后是知名度更高的 HSE06 泛函，最后在不得已的情况下再尝试修正能力最强、但也最耗时的 GW 方法。

## mBJ 泛函

参见 [VASP软件包mBJ能带结构计算指南](<https://mp.weixin.qq.com/s/4xyhq0vR6E3gt-BcuSOdDw>)，非常全面，这里仅做一些补充说明
### K 点选取
因为 mBJ 泛函收敛非常耗时，像 PBE 泛函那样，收敛到基态后，再使用包含能带路径 K 点的 KPOINTS 做非自洽会耗费大量时间。因此我们需要在自洽计算的 KPOINTS 中就加入能带路径的 K 点，但将权重设为 0，这样既可以避免这些非均匀分布的 k 点对基态能量判断产生干扰，又可以在输出的 EIGENVAL 中直接得到这些点的能量本征值。
使用 vaspkit 的 303 或 302 功能先生成能带路径文件 KPATH.in，再利用 251 功能将均匀的 k-mesh 与能带路径 k 点合并成一个 KPOINTS 文件

```bash
vaspkit -task 303
vaspkit -task 251
```

### 自洽法

先做一个 PBE 自洽，得到收敛的基态，注意这里已经需要使用包含权重为 0 的 k 点的 KPOINTS 。
然后读入 PBE 自洽的基态波函数（不读取电荷密度）进行 mBJ 计算，这样可以显著加快收敛速度。
mBJ 计算不支持对 K 点并行，所以只能通过 NCORE 设置并行参数，不能使用 KPAR。

```fortran
ISTART  = 1
NELM    = 300
METAGGA = MBJ
LASPH   = .TRUE.
```

注意 mBJ 计算会导致系统总能量大于 0，这是正常的。

### 微扰法

先进行 PBE 自洽，然后进行单步的 mBJ 自洽

```fortran
ISTART = 1
NELM   = 1

METAGGA = MBJ
LASPH   = .TRUE.

# NELMDL=1 # This can improve the convergence speed in VASP.6 but is not recommended in VASP.5.
```

在 OUTCAR 中得到迭代后的修正系数 CMBJ

```bash
grep CMBJ OUTCAR
```

在 INCAR 中手动固定参数 CMBJ，做一步精确对角化 (注意精确对角化非常吃内存，一般只有小体系才能用)

```fortran
ALGO =EXACT
CMBJ =
```
## HSE06 泛函
### K 点选取
和 [[4 Beyond PBE#mBJ 泛函|mBJ泛函]]相同
### 自洽计算
先做一步 PBE 自洽
因为 HSE 自洽很难收敛，所以取 ALGO=Damped，减少收敛精度至 1e-5。使用 ISTART=1 读入上一步 PBE 自洽的波函数

```fortran
ISTART = 1 

HSE06 Calculation

ALGO = Damped
TIME = 0.5
LHFCALC = .TRUE.
GGA = PE
HFSCREEN = 0.2
EDIFF = 1E-05
```
### 罕见错误

```fortran
internal error in SET_INDPW_FULL: insufficient memory (see wave.F safeguard)
```
~~这并非真正的系统内存不足，而是 VASP 本身在预分配波函数内存空间时，给出的一个非常保守的警示。使用 VASP.6.3.0 可以避免这一问题~~，K 点过多也可能会导致这一问题

## 其他杂化泛函

杂化泛函，作为一种广为接受的更加贴合实验的 DFT 计算手段，根据各个杂化项的来源以及混合参数的不同，特别在量子化学领域，已经发展出了非常多的种类。VASP 原生支持的杂化泛函，见 [Hybrid functionals: formalism - VASP Wiki](https://www.vasp.at/wiki/index.php/Hybrid_functionals:_formalism)，也有好几种。但当各种参考资料讨论用杂化泛函修正固体的带隙时，几乎就固定为 HSE 泛函这唯一一种选择，原因何在？

从 VASP 对杂化泛函的分类依据中：
1. Unscreened hybrid functionals
2. Range-separated hybrid functionals
    1. Error function screening with short-range Hartree-Fock exchange
    2. Error function screening with long-range Hartree-Fock exchange
    3. ...
我们可以初窥端倪。它们的主要区别，在于 Hartree-Fock（HF）交换能这一项中，是包含全部能量，还是仅包含短程作用能量或长程作用能量。

HF 近似对于电子交换作用的描述，基于库伦相互作用，这是一种在长程下依然存在的相互作用。但在固体中，存在着屏蔽效应 (screening effect, or shielding effect) ，这里直接摘抄百科对屏蔽效应的解释

> 在多电子原子中，一个电子不仅受到原子核的引力，而且还要受到其他电子的排斥力。内层电子排斥力显然要削弱原子核对该电子的吸引，可以认为排斥作用部分抵消或屏蔽了核电荷对该电子的作用，相当于使该电子受到的有效核电荷数减少了

因此，对于固体中的外层价电子而言，它们所感受到的其实是一种“削弱的”、短程的相互作用，即屏蔽库伦势（screened Coulomb potential）。而固体能带的带隙，正是主要由价电子所贡献的。所以，在固体计算中，使用仅考虑短程 HF 能量的 HSE 泛函，可以在节省计算时间的情况下保持精度不变。包含全 HF 能量的 B3LYP 等泛函，则更适合分子计算。
## GW

G 代表格林函数，W 代表屏蔽库伦势

因为 GW 计算过程较为复杂，在正式开始 GW 计算前，有几点需要注意的地方：

1. 使用专门的 GW 赝势！参照[推荐赝势表](https://www.vasp.at/wiki/index.php/Available_PAW_potentials#Recommended_potentials_for_GW.2FRPA_calculations ) 
2. 以下教程是基于 VASP. 6.3.2 版本进行的。VASP. 6.x.x 版本对 GW 计算模块做了较多改动，参数设置与流行的 VASP. 5.4.4 版本有所不同，因此如果需要在 VASP. 5.4.4 版本上执行 GW 计算，务必参照 [官方手册](https://www.vasp.at/wiki/index.php/Practical_guide_to_GW_calculations) 修改 INCAR 中 ALGO、NELMGW 等参数。
3. 使用相同核数并行的情况下， GW 计算时 CPU 温度相比于常规 PBE 自洽计算时明显更高，因此在本地机计算时要时刻留心。
4. HSE 和 GW 方法，对带隙的修正效果是可以叠加的，因此可以从一个 HSE06 泛函的自洽基态出发做 GW 计算。**但尚不清楚这样做是否会引入 double counting 问题**
5. VASP. 6.x.x 开始提供的单步 GW 方法，在流程大幅度简化的同时，结果的稳定性也较强，因此，如果单步 GW 计算没有出现 bug，且结果已足以与实验吻合，就无需再使用复杂的多步 GW 方法。具体 INCAR 设置参见 [官方手册](https://www.vasp.at/wiki/index.php/Practical_guide_to_GW_calculations)。
6. GW 方法对 Mott 绝缘体的描述学界尚有争议。金属化合物还是优先考虑简单的 [[0 输入文件#DFT+U|DFT+U]] 方法
7. 使用 GW 赝势后，PBE 或 HSE 自洽时，少数情况下会出现莫名其妙的内存溢出错误，可以尝试注释掉 NCORE

### 1 空带生成
因为 GW 计算更为耗时，已经无法通过在 KPOINTS 里加入权重为 0 的 k 点的方法间接计算能带了，所以 KPOINTS 设置为常规的均匀 k-mesh。
GW 计算耗时随 k 点密度的增长是非线性的，所以保证精度前提下，一定要选尽可能少的 k 点。k 点密度对 GW 能带结构的影响，与 PBE 是类似的，所以精度检查可以通过算 PBE 能带的方式来高效的进行。
GW 计算需要非常多的额外空带，这里从 **HSE06 收敛基态** 出发（参见上一节的计算流程），将能带数目增大约2个量级至1024，做单轮精确对角化以得到准确的空带本征态。
打开 LOPTICS 的目的是为了生成 WAVEDER 文件，得到轨道相对于波矢 k 的导数，以加速后续计算。
```fortran
ISTART = 1 
LREAL = .FALSE.
LWAVE = .TRUE.
LCHARG = .TRUE.
ADDGRID= .TRUE.

ISMEAR = 0
SIGMA = 0.05 
# EDIFF = 1E-08 
NCORE = 4  

HSE06 Calculation

# ALGO = Damped
TIME = 0.5
LHFCALC = .TRUE.
GGA = PE
HFSCREEN = 0.2
# EDIFF = 1E-05

Unoccupied States

LOPTICS = .TRUE.
ALGO = Exact
NELM = 1
NBANDS = 1024
```
### 2 GW 计算

这里 GW 算法选择部分自洽的 EVGW0，它在大多数时候与实验吻合的最好。迭代轮数取 4 轮也是在精度和算力之间取得一个比较好的平衡。注意 GW 方法不支持 NCORE 或其他并行参数，所以需要将这一行注释掉
`PRECFOCK = Fast` 对小体系提速很明显，约 30%左右，产生的误差在 meV 量级，对能带整体结构没有影响，可以放心使用
```fortran
ISTART = 1 
LREAL = .FALSE.
LWAVE = .TRUE.
LCHARG = .TRUE.
ADDGRID= .TRUE.

ISMEAR = 0
SIGMA = 0.05 
# EDIFF = 1E-08 
# NCORE = 4  

NBANDS = 1024
NELMGW = 4   ! use NELM for VASP.6.2 and older
ALGO = EVGW0 ! use "GW0" for VASP.5.X 
PRECFOCK = Fast
```
### 3 Wannier 拟合
GW 计算完成后，如果我们观察 EIGENVAL 文件，可以发现在多体修正下，能带的本征值已经不是按照从小到大的规律排列了。因此，无论是仅关心带隙，还是想得到 GW 修正后的能带，都必须要拟合 Wannier 函数。VASP. 6.x.x 开始支持 [[SCDM Wannier]] 方法，在不关心体系对称性的情况下，这一方法确实能够非常快速方便的拟合得到光滑的能带，但实际测试发现它对带隙的处理并不是非常精确，因此首先还是应该尝试常规设置投影子的方法。
`exclude_bands = 65-1024` 这里建议将多余的空带排除，剩余比 `NUM_WANN` 略多一些的能带即可，否则 wannier 拟合过程会消耗非常多的机时
```fortran
ISTART = 1 
LREAL  = .FALSE.
LWAVE  = .TRUE.
LCHARG = .TRUE.
ADDGRID= .TRUE.

ISMEAR = 0
SIGMA  = 0.05 
# EDIFF = 1E-08 
# NCORE = 4  

NBANDS = 1024
ALGO = NONE
NELM = 1
  
NUM_WANN=48
LWANNIER90_RUN=T
LWRITE_MMN_AMN = .TRUE.

WANNIER90_WIN="
exclude_bands = 65-1024
write_hr=.true.
dis_num_iter=200
num_print_cycles=200
num_iter=1000
guiding_centres=.true.
write_xyz=.true.
use_ws_distance=.false.

begin projections
C:s,p
end projections
"
```
