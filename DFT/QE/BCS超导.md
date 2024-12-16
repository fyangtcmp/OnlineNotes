==经验表明，各种精度上的误差都更倾向于使 Tc 大于实际值，需要对计算结果十分小心，以免出现“高温超导”的乌龙！！==

QE研究材料超导性质，是先通过DFPT理论计算电子-声子耦合系数，然后在经典BCS超导理论框架下计算超导转变温度Tc。

其中，电子-声子耦合系数的计算有两种方法，第一类是通过PHonon模块直接计算，详细流程可参见：

[QE实践详解 - DFTbook (yyyu200.github.io)](https://yyyu200.github.io/DFTbook/blogs/2019/04/01/HandsOn/#43-%E7%94%B5%E5%AD%90-%E5%A3%B0%E5%AD%90%E8%80%A6%E5%90%88%E7%B3%BB%E6%95%B0 "QE实践详解 - DFTbook (yyyu200.github.io)")

但是该方法计算耗时十分惊人，因此对于大体系晶胞，比较可行的是通过第二类方法，也就是EPW (Electron-Phonon Wannier) 模块计算，官方主页：

[About — EPW https://gitlab.com/pages/sphinx documentation (epw-code.org)](https://docs.epw-code.org/doc/About.html "About — EPW https://gitlab.com/pages/sphinx documentation (epw-code.org)")

不过，第二类算法“耗时更少”也只是相对而言的，建议全程在超算上进行。以下流程将以3DGDY在天河二号超算上的作业提交代码为示例。注意yh系列和slurm系列作业管理系统虽然有继承关系，但具体操作起来有很关键的不同，比如以下代码使用的是yhrun，但在slurm系统上使用srun就会报错，反而要使用mpirun。

以下带有npool指令的pw自洽并行步骤，都需要在运行前额外测试不加npool是否会报FFT错误。可能的错误信息包括：FFT所用到的planes数目少于核数、某些核没有分配到FFT plane等。这种时候，如果加入 npool指令，qe会不输出错误信息强行计算，导致电子步计算时长显著增长20~50倍。此时需额外加入

```bash
pw.x -pd .true.
```

用pencil decomposition代替默认的slab decomposition，并去除-npool 96指令。参见[https://www.mail-archive.com/users@lists.quantum-espresso.org/msg41493.html](https://www.mail-archive.com/users@lists.quantum-espresso.org/msg41493.html "https://www.mail-archive.com/users@lists.quantum-espresso.org/msg41493.html")。epw并行依然使用npool。

## 流程与校验

![[图片 1_X8ly6_0mnT.png]]
## 1.电子自洽计算

常规静态自洽计算，采用比较密的k-mesh，最好是之后非自洽k-mesh的两倍以上。如果体系本身较大，或者机时不允许，也需要保持自洽非自洽k-mesh的奇偶性一致，设置错误会对最终结果影响明显！

展宽方式smearing='mp'，虽然mp展宽只适用于金属，但超导体常态必须是金属，所以无关紧要。

自洽电子步一般比较快，这里就不对输出做重定向，直接交互式运行

```bash
yhrun -p gscomp -n 24 pw.x < scf.in
```

计算完成后，将结果复制到几个独立的文件夹，分别用于能带计算、费米面计算、声子计算和非自洽计算。

```bash
cd ..
cp -r scf fermi
cp -r scf band
cp -r scf phonon
cp -r scf nscf

```

## 2.电子非自洽计算

非自洽计算，采用稀疏的 (coarse) ，且各向同性的k-mesh。非自洽计算相比于自洽计算，耗时很少，但对epw后续插值影响很大，所以k-mesh也不能过于稀疏，如6x6x6是比较合适的

qe的非自洽计算需要明确写出每个k点的坐标和权重，在wannier90-3.1.0/utility/目录下，使用

```bash
kmesh.pl 6 6 6 >> nscf.in
```

将均匀k-mesh中每一个点的相对坐标追加至nscf.in。

epw计算，直接读入的电子成分就是非自洽产生的电子波函数。因为qe-7.0版的EPW不支持对于同一个波函数存储文件的并行计算，所以在并行时，需要使用-npool指令将波函数按照总并行核数拆分成一一对应的关系，如：

```bash
yhrun -p gscomp -n 96 pw.x -npool 96 < nscf.in
```

且后续epw计算的并行核数需要和这一步等同。因为epw计算比较耗时，所以核数越多越好，但注意核数不能高于kmesh.pl产生的k点总数。

## 3.声子计算

声子计算，q-mesh需要是第2步中k-mesh的整数因子，如3x3x3或6x6x6。

这一步耗时、耗硬盘最为严重，3DGDY取3x3x3 q-mesh，tr2_ph = 1.0d-17高精度，96核并行需要接近一周，占据超过100G的硬盘空间。而且根据经验来说，3x3x3 q-mesh还是过于粗糙，后续精确计算时需要提升至6x6x6。

同时，声子计算耗时受自洽k-mesh精度影响也很明显，如果耗时难以接受，无奈之举是放弃之前自洽电子步的结果，单独做一个稀疏格点的自洽电子计算对接声子计算。

输入文件ph.in示例如下，使用yhbatch提交一般就不需要recover指令了：

```bash
&inputph
 prefix   ='<prefix>',
 fildyn   ='<prefix>.dyn.xml',
 fildvscf = 'dvscf',
 tr2_ph   =  1.0d-17
 ! recover  = .true.,
 ldisp    = .true.,
 nq1 = 3, nq2 = 3, nq3 = 3
 /
```

存疑：epw官网教程，要求声子步也采用和电子非自洽步相同的核数运行，但实际测试发现，声子步的核数并不会影响结果。且epw程序并不会读入声子计算所产生的波函数，所以改变核数应当不违反epw不能对波函数并行的基本原则。

虽然吕梁超算中心给出的默认设置核数-n是节点数-N的24倍，但实际执行时会有一定的问题，建议对核数有精确要求时依然使用-n而不是-N。-npool设为96也是为了提高并行效率，并不是强制要求。将下列内容保存成脚本，使用yhbatch提交

```bash
#!/bin/bash
yhrun -p gscomp -n 96 ph.x -npool 96 < ph.in > ph.out
```

如果使用nohup yhrun提交，任务容易被超算kill

在log文件ph.out中，搜索'CPU'（大写）可以查看每个q点的计算时长（WALL time），以此估算总时长，不过要注意Gamma点计算一般会更快一些，因此要用第二个不等价q点估算。搜索'freq'可以查看每个q点的频率，可观察Gamma点是否有虚频。

计算完成后，得到一系列dyn文件，如

```text
<prefix>.dyn0
<prefix>.dyn1.xml
<prefix>.dyn2.xml
...
```

dyn0是纯文本文件，包含了q-mesh尺寸，不可约q点的数目和位置。dyn1、dyn2……是xml格式文件，给出了具体每个不可约q点的声子计算结果。因为我们在ph.in中设置了dyn文件后缀为xml，所以这里会自动加上xml，虽然这样做更加符合文件的内部存储形式，但在后续对接声子谱计算时就产生了问题。声子谱计算的q2r.x需要读入文件名统一的dyn文件，因此我们将q2r.in设置为

```bash
&input
 fildyn='<prefix>.dyn.xml', zasr='simple', flfrc='<prefix>.fc.xml'
 /
```

然后

```batch
cp  <prefix>.dyn0  <prefix>.dyn0.xml
yhrun -p gscomp -n 1 q2r.x < q2r.in > q2r.out
```

运行程序将动力学矩阵从q空间变换到实空间。再编辑输入文件matdyn.in

```bash
&input
  asr='simple',
  amass(1)=12.0107,
  flfrc='<prefix>.fc.xml',
  flfrq='<prefix>.freq.dat',
  q_in_band_form=.true.,
  q_in_cryst_coord=.true.,
/
6
0.0         -0.5         0.5     30
0            0           0       30
0.3805       0.3805      0.3805  30
0.5          0.5         0.5     30
0.5          0.0         0.0     30
0.0          0.0         0.0     1
```

使用

```纯文本
yhrun -p gscomp -n 1 matdyn.x < matdyn.in > matdyn.out
```

得到特定路径上的声子谱，输出文件.freq.dat的格式和能带计算band.x输出一致，检查其是否存在虚频。确认无误后，使用QE/EPW/bin中的pp.py程序，将EPW计算所需的信息按照不可约q点整理至一个新的子文件夹。

## 4.EPW计算

参考我们在一开始给出的流程图，EPW程序内部，还可以继续分为5个子程序。我们既可以选择在单次计算中跑完整个EPW程序，也可以将计算拆分为几个部分，先检验**2.电子非自洽计算**、**3.声子计算**的精度是否合适，再进行电声耦合计算，最后进行后续物理性质的计算。运行脚本为

```bash
#!/bin/bash
yhrun -p gscomp -n 96 epw.x -npool 96 < epw0.in > epw0.out
yhrun -p gscomp -n 96 epw.x -npool 96 < epw1.in > epw1.out
yhrun -p gscomp -n 96 epw.x -npool 96 < epw2.in > epw2.out

```

再次注意并行核数必须和非自洽计算步等同，而且-npool是强制需求。运算时间较长，采用yhbatch提交。重复计算时，可根据需求注释掉不需要的步骤，比如epw0.in这一步。

首先是对2、3两步进行校验的输入文件 epw0.in

```
&inputepw
  prefix      = '3DGDY',
  amass(1)    = 12.0107
  outdir      = './'
  dvscf_dir   = '../phonon/save'
  iverbosity  = 2

  ep_coupling = .true.  
  elph        = .true.
  epwwrite    = .true.
  epwread     = .false.

  wannierize  = .true.
  num_iter    = 500

  auto_projections = .true.
  scdm_proj        = .true.
  scdm_entanglement= gaussian
  scdm_mu          = 2.4624
  scdm_sigma       = 1.0
 
  efermi_read = .true.
  fermi_energy= 2.4624
  fsthick     = 0.4 ! eV
  degaussw    = 0.025 ! eV
  
  band_plot   = .true.
  filkf = './epw_kpath.dat'
  filqf = './epw_kpath.dat'

  nk1         = 6
  nk2         = 6
  nk3         = 6

  nq1         = 3
  nq2         = 3
  nq3         = 3
/
```
然后是电声耦合计算的输入文件 epw1.in

```
&inputepw
  prefix      = '3DGDY',
  amass(1)    = 12.0107
  outdir      = './'
  dvscf_dir   = '../phonon/save'
  iverbosity  = 2

  ep_coupling = .true.  
  elph        = .true.
  epwwrite    = .false.
  epwread     = .true.
  
  ephwrite    = .true.

  wannierize  = .false.
 
  efermi_read = .true.
  fermi_energy= 2.4624
  fsthick     = 0.4 ! eV
  degaussw    = 0.025 ! eV

  mp_mesh_k = .true.
  nkf1         = 40
  nkf2         = 40
  nkf3         = 40

  nqf1         = 20
  nqf2         = 20
  nqf3         = 20

  nk1         = 6
  nk2         = 6
  nk3         = 6

  nq1         = 3
  nq2         = 3
  nq3         = 3
 /
```
最后是求解物理性质，也即超导转变温度的输入文件 epw2.in

```
&inputepw
  prefix      = '3DGDY',
  amass(1)    = 12.0107
  outdir      = './'
  dvscf_dir   = '../phonon/save'
  iverbosity  = 2

  ep_coupling = .false.  
  elph        = .false.
  epwwrite    = .false.
  epwread     = .true.
  
  ephwrite    = .false.

  wannierize  = .false.
 
  efermi_read = .true.
  fermi_energy= 2.4624
  fsthick     = 0.4 ! eV
  degaussw    = 0.025 ! eV

  eliashberg  = .true.
  muc     = 0.1 

  mp_mesh_k = .true.
  nkf1         = 40
  nkf2         = 40
  nkf3         = 40

  nqf1         = 20
  nqf2         = 20
  nqf3         = 20

  nk1         = 6
  nk2         = 6
  nk3         = 6

  nq1         = 3
  nq2         = 3
  nq3         = 3
/
```

接下来我们将按顺序介绍输入文件中参数的作用

### Wannier拟合

```bash
  wannierize  = .true.
  num_iter    = 500

  auto_projections  = .true.
  scdm_proj         = .true.
  scdm_entanglement = gaussian
  scdm_mu           = 2.4624
  scdm_sigma        = 1.0

```

对于一些必要的wannier90关键词，epw提供了直接写入的接口，其他非必要的关键词可以通过wdata写入。这里展示的，是epw原生支持、但教程中没有涉及的SCDM方法拟合wannier函数，传统拟合Wannier函数的方法官方文档展示了很多，这里就不在赘述。

### Wannier插值校验

epw方法的思想，是使用Wannier函数，将稀疏格点的DFT计算结果插值至密格点，基于密格点求解实际物理性质，从而减轻计算量。因此，插值的好坏就决定了我们计算结果的好坏。首先要比较的就是Wannier函数插值出的电子能谱和声子能谱，和DFT结果是否吻合，使用

```bash
band_plot   = .true.
filkf       = 'epw_kpath.dat'
filqf       = 'epw_kpath.dat'
```

输入路径文件的格式参见 epw_kpath.dat

```Bash
151.000000  cartesian
 -0.574473  -0.331672  0.000000  1.0
 -0.555324  -0.320617  0.000000  1.0
 -0.536175  -0.309561  0.000000  1.0
 -0.517026  -0.298505  0.000000  1.0
……
```

然后，因为Wannier函数是实空间的一个局域化函数，所以插值计算得到的各项物理量，也应当是随着实空间距离指数衰减的。epw会输出

```bash
decay.H
decay.dynmat
decay.epmate
decay.epmatp
```

这4个文件，用以检查哈密顿量、动力学矩阵、电声耦合矩阵电子部分、声子部分的衰减性，引用epw官方教程中的图片

![[image_3PTzqhF6Ad.png]]
可见，这里电子部分衰减了5个量级，说明精度已足够，但声子部分只衰减了2个量级，所以还需要继续提高精度。

### 电声耦合矩阵 (coarse mesh)

```bash
nk1         = 6
nk2         = 6
nk3         = 6

nq1         = 3
nq2         = 3
nq3         = 3
```

epw 将在我们稀疏格点非自洽计算和声子计算的基础上，计算电声耦合矩阵，所以这里的 nk 和 nq 要和之前的计算相匹配。

### 电声耦合矩阵 (dense/fine mesh)

将稀疏格点的电声耦合矩阵（还有哈密顿量等等），通过插值方法变换到密格点中

```bash
mp_mesh_k = .true.
nkf1         = 40
nkf2         = 40
nkf3         = 40

nqf1         = 20
nqf2         = 20
nqf3         = 20
```

这里 nqf 是 nkf 的整数因子。使用 mp_mesh_k 将只计算不等价 k 点从而减轻计算量，mp_mesh_q 虽已在官方文档中列出，但尚不可用，因此对 q 点的计算依然需要遍历 mesh 中的每一个点，通过设置

```bash
fsthick     = 0.4
```

epw 程序将预先计算**k**电子被 **q** 声子散射后的 **k+q**态是否落在 fsthick 所规定的费米面附近的能量窗口内，以此筛除对电声耦合矩阵无贡献的 q 点以减轻计算量。这一能量窗口截断的做法是对声子线宽公式

![[image_9jnu3IeTfL.png]]
中 delta 函数的近似。因此 fsthick 不能取太大，一般0.4足以。3DGDY 测试将 fsthick 取为1.0后占用内存达到364GB，已经超过了超算可用的内存空间。

如果后续想提高 mesh 精度，官方并未给出标准工作流程。直接在同一文件夹下改输入文件重新计算会报错，可尝试删除 restart. fmt 文件，但尚不清楚会不会引入其他问题。和 DFT 计算一样，提高 k-mesh 带来的额外计算代价很少，但对精度改善是很大的。经过测试，nkf 从20提高到40，对参考文献的结果有显著的逼近。关于调节 mesh 大小，更多内容参见

[https://www.jianshu.com/p/e5e34d576c86](https://www.jianshu.com/p/e5e34d576c86 "https://www.jianshu.com/p/e5e34d576c86")

超导转变温度对费米面非常敏感，因此传入高精度非自洽计算的费米面

```bash
efermi_read  = .true.
fermi_energy = 6.8551
```

同时，epw也会给出一个基于插值计算的费米面估计值，注意观察这两个值是否有显著差别，以排查拟合的wannier函数是否还有隐藏的问题。

```
epwwrite    = .true.
epwread     = .false.
```

将Wannier表象哈密顿量、电声耦合矩阵等信息存盘。按照旧官方文档的说法，后续读盘时需要加入额外的一个关键词 _kmaps_，但2021年官方在线课程的范例文档已经不使用_kmaps_了，实际测试也发现加入反而会报错，最新版只用这两个关键词就足以。

```
ephwrite    = .true.
```

将 Bloch 表象电声耦合矩阵等信息存入文件夹\<prefix\>.ephmat。没有对应的\_ephread\_关键词，epw 程序会自动读入\<prefix\>.ephmat 文件夹。因为经过一次插值，变换后数据量将暴涨3个量级。所以，依然要随时检查硬盘空间是否富余。

### 物理性质计算

\<prefix\>.ephmat 文件夹内包含了求解 Eliashberg 方程所需的所有信息。因此，求解物理性质时，我们可以将电声耦合计算和写入全部关闭，仅读入已存盘的数据。见输入文件 epw2.in

```
ep_coupling = .false.  
elph        = .false.

epwwrite    = .false.
epwread     = .true.
ephwrite    = .false.
```

最后，启动 Eliashberg 方程的计算

```
eliashberg  = .true.
muc = 0.1
```

如果只关心超导转变温度，就无需再设置其他参数了。

muc为有效屏蔽的库仑排斥常数，根据参考文献取经验值0.1，epw官方教程一般也取在0.1附近。