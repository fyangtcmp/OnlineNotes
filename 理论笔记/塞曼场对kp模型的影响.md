## 背景知识
拓扑物理研究中，有三类常用的模型，分别是：
1. 紧束缚模型（tight-binding model，以下简称 tb 模型），例如
$$
H=\sum_{ij}t_{ij}c^{\dagger}_i c_j
$$
2. 晶格模型（lattice model），例如
$$
H=t_1 \cos(k_x)\sigma_x+t_2 \sin(2k_x)\sigma_z+\cdots
$$
3. $k\cdot p$ 模型（以下简称 kp 模型），例如
$$
H=a k_x \sigma_x+b k_y \tau_z\sigma_y+c k_x^2 \tau_y+\cdots
$$
这里 $\sigma,\tau$ 是泡利矩阵，代表自旋空间和轨道空间（具体对应关系以及写法的先后顺序，并没有统一约定），$\sigma_0,\tau_0$ 项一般略去不写，如果补充完整的话，上面的哈密顿量，等价于
$$
H=a k_x \tau_0\sigma_x+b k_y \tau_z\sigma_y+c k_x^2 \tau_y\sigma_0+\cdots
$$

tb 模型和晶格模型，是完全等价的，它们可以通过傅里叶变换相互转换，因此它们有时候也可被统称为 tb 模型。tb 模型通过在倒空间中某一点附近做级数展开，也可以得到 kp 模型。但是 kp 模型如果想变换回 tb 模型，就只能通过“比较”kp 模型中每一项和三角函数的展开项，做一个形式上的转换。这种转换也只在级数展开的参考点上是严格的。

## 塞曼场
在模型研究中，一般用塞曼场项来模拟外加磁场的效应
$$
H_{Zeeman}=\mu_B\vec{B}\cdot\vec{\sigma}
$$
考虑一个简单的二带 kp 模型（略去常数系数）
$$
H_{\mathrm{kp}}^0= v_1 (k_x \sigma_x + k_y\sigma_y) + v_2(k_x^2+k_y^2) 
$$
加上 x 方向的磁场，得到
$$
\begin{align}
H_{\mathrm{kp}}=&H^0_{\mathrm{kp}}+H_{Zeeman}\\
=&v_1(k_x \sigma_x + k_y \sigma_y) + v_2(k_x^2+k_y^2) + \mu_B B_x\sigma_x\\
=&v_1(k_x+\mu_B B_x/v_1) \sigma_x + v_1 k_y \sigma_y\\
&+ v_2 [(k_x+\mu_B B_x/v_1)^2-2\mu_B B_x/v_1\cdot k_x-(\mu_B B_x/v_1)^2]\\
&+v_2 k_y^2\\
=&v_1\widetilde{k_x} \sigma_x + v_1 k_y \sigma_y+ v_2\widetilde{k_x}^2+v_2k_y^2 
-2\mu_B B_x v_2/v_1\cdot \widetilde{k_x} -v_2(\mu_B B_x/v_1)^2
\end{align}
$$
其中，$v_2(\mu_B B_x/v_1)^2$ 是加在主对角线上的常数项，可以略去。比较剩余项的形式，我们可以发现多出来一个 $-2\mu_B B_x v_2/v_1\cdot\widetilde{k_x}$ 项，它代表着外加塞曼场，不仅将 Dirac 锥的极值点从 $k_x$ 点移动到 $\widetilde{k_x}$ 点，在模型存在二阶项的情况下，还会额外的使 Dirac 锥产生倾斜

参考文献：[Origin of planar Hall effect on the surface of topological insulators: Tilt of Dirac cone by an in-plane magnetic field (aps.org)](https://journals.aps.org/prb/pdf/10.1103/PhysRevB.101.041408)