## 简介
根据一个群的对称性，给出一个张量在这个群的所有对称性约束下的形式，包括有哪些禁止的分量，哪些允许的分量，哪些分量是等价的
点群和空间群：https://www.cryst.ehu.es/cgi-bin/cryst/programs/tensor.pl
磁点群和磁空间群：https://www.cryst.ehu.es/cgi-bin/cryst/programs/mtensor.pl
参考文献：https://doi.org/10.1107/S2053273319001748
## 使用方法
建立自定义张量基于的是 Jahn's symbol，具体记号法则可以在 https://www.cryst.ehu.es/html/cryst/mtensor_help.html 查阅，这里只提一个容易迷惑的点，那就是轴张量（axial tensor）的首字母是 a，但在 Jahn's symbol 里的代表字母是 e。时间反演下反号在 Jahn's symbol 里才是用字母 a 代表的。
### 例 1：BCD 贡献的非线性霍尔电导
定义式为
$$
j_a=\chi_{abc}(\tau) E_b E_c
$$
其中电导率包含弛豫时间的一次项。
我们先分析这个公式在时间反演操作下的表现，电流密度在经典视角下，是电荷量与电子速度的乘积，而速度是对时间的导数，所以电流密度在时间反演下会反号。弛豫时间同样也会反号，因为它概念上来自于电子的散射运动。电场强度则保持不变。因此我们可以得到
$$
\begin{align}
\mathcal{T}\left(j_a\right)\mathcal{T}^{-1}&=-j_a\\
\mathcal{T}\left[\chi_{abc}({\tau}) E_b E_c\right]\mathcal{T}^{-1}&=\chi_{abc}(-\tau) E_b E_c
\end{align}
$$
显然，欲使等式左右侧依然成立，这个电导率张量在时间反演操作下保持不变即可。
然后我们分析这个电导率在非正当转动下的表现。因为 $j_a, E_b, E_c$ 均为极矢量，所以 $\chi_{abc}(\tau)$ 也只能是极矢量。
汇总以上结果，可以得到 $\chi_{abc}(\tau)$ 在 Jahn's symbol 中的形式为 `V3`
### 例 2：本征非线性平面霍尔效应
定义式为
$$
j_a=\chi_{abcd} E_b E_c B_d
$$
这里多出来了一个新的矢量，磁场矢量。根据经典力学的毕奥-萨伐尔定律，磁场来源于元电流矢量和单位矢量的叉乘，因此这是一个赝矢量，而等式左侧的电流是极矢量，因此为了和磁场相抵消，$\chi_{abcd}$ 只能也是赝矢量
并且因为元电流在时间反演操作下反号，就如我们例 1 中提到的，所以磁场也会在时间反演操作下反号。据此我们可以写出
$$
\begin{align}
\mathcal{T}\left(j_a\right)\mathcal{T}^{-1}&=-j_a\\
\mathcal{T}\left(\chi_{abcd} E_b E_c B_d\right)\mathcal{T}^{-1}&=-\chi_{abcd} E_b E_c B_d
\end{align}
$$
等式左右两侧依然是成立的，所以 $\chi_{abcd}$ 在时间反演作用下保持不变。汇总以上结果，可以得到 $\chi_{abcd}$ 在 Jahn's symbol 中的形式为 `eV4`

### 例 3：自旋霍尔效应
定义式为
$$
j^c_a=\sigma_{ab}^c E_b
$$
这个例子稍微复杂一点，因为这里上角标的 $c$ 是自旋指标，等式左侧的 $j_a^c$ 代表自旋极化的电流。我们这里采用一种不一定严谨的方法，将 $j_a^c$ 理解为电流和自旋极化的乘积，如
$$
j_a^c\sim j_a s_c
$$
对于自旋极化，我们也可以从经典图像出发，即电子自旋运动所产生的元电流导致的磁矩，所以在时间反演对称性作用下，它应该和磁场矢量有类似的行为，因此
$$
\begin{align}
\mathcal{T}\left(j_a^c\right)\mathcal{T}^{-1}&=\mathcal{T}\left(j_a s_c\right)\mathcal{T}^{-1}=(-j_a)(-s_c)=j_a^c\\
\end{align}
$$
显然等式右侧的 $\sigma_{ab}^c$ 也应当是时间反演不变的。又因为自旋极化 $s_c$ 是赝矢量，所以等式右侧的 $\sigma_{ab}^{c}$ 也需要是赝张量。汇总得到 $\sigma_{ab}^{c}$ 在 Jahn's symbol 中的形式为 `eV3`

### 汇总表
除了 `a` 和 `e` 记号以外，Jahn's symbol 本身还支持张量指标交换对称反对称性的描述，大括号代表指标交换反对称，中括号代表指标交换对称。

|               Type | Jahn's symbol | Reference                |
| -----------------: | ------------: | ------------------------ |
|      intrinsic AHC |         a{V2} | *RevModPhys. 82.1539*    |
|  intrinsic 2nd AHC |        a{V2}V | *PhysRevLett.127.277202* |
|  intrinsic 3rd AHC |     a{V2}[V2] | *PhysRevB.107.075411*    |
|      extrinsic AHC |          {V2} |                          |
|  extrinsic 2nd AHC |         {V2}V | *PhysRevLett.115.216806* |
|  extrinsic 3rd AHC |        {V2}V2 | *PhysRevB.105.045118*    |
|                    |               |                          |
|      intrinsic PHC |        e{V2}V | *PhysRevLett.132.056301* |
|  intrinsic 2nd PHC |       e{V2}V2 | *PhysRevLett.130.126303* |
|  extrinsic 2nd PHC |      ae{V2}V2 | *PhysRevB.108.075155*    |
|                    |               |                          |
|      intrinsic SHC |           eV3 | *RevModPhys. 87.1213*    |
|  intrinsic 2nd SHC |           eV4 | *PhysRevLett.134.056301* |
|      extrinsic SHC |          aeV3 |                          |
|                    |               |                          |
|               CISP |          aeV2 |                          |
| intrinsic 2nd CISP |          aeV3 | *PhysRevLett.129.086602* |
| extrinsic 2nd CISP |           eV3 | *PhysRevLett.130.166302* |

