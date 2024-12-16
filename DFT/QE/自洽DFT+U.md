## 参考文献
[https://indico.cern.ch/event/611331/contributions/2645475/attachments/1511261/2356868/Presentation_Timrov_625.pdf](https://indico.cern.ch/event/611331/contributions/2645475/attachments/1511261/2356868/Presentation_Timrov_625.pdf "https://indico.cern.ch/event/611331/contributions/2645475/attachments/1511261/2356868/Presentation_Timrov_625.pdf")

[https://journals.aps.org/prb/abstract/10.1103/PhysRevB.71.035105](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.71.035105 "https://journals.aps.org/prb/abstract/10.1103/PhysRevB.71.035105")

[https://journals.aps.org/prb/abstract/10.1103/PhysRevB.98.085127](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.98.085127 "https://journals.aps.org/prb/abstract/10.1103/PhysRevB.98.085127")

## 流程

旧版QE的hp.x模块在计算六角晶格时可能有收敛问题，建议采用最新的v6.7版本。

目前DFPT法和传统的线性响应法求U仅适用于open-shell system，即系统中有未成对电子的情况。奇数电子系统必然是开壳层系统

1. 准备QE输入文件：
    
    使用在线小程序 [http://www.densityflow.com/p2p.php](http://www.densityflow.com/p2p.php "http://www.densityflow.com/p2p.php") 可直接从POSCAR得到QE输入文件，然后进行修改。因为不需要做结构弛豫，可将&control部分 tprnfor 等参数删除。对于绝缘体，需要删除occupations一行，使用默认的无展宽模式。
    
    然后在&system中添加DFT+U参数： lda_plus_u= .true., lda_plus_u_kind= 0, U_projection_type= 'ortho-atomic', Hubbard_U(i) =1.d-8 将待求元素的所有原子放在到ATOMIC_POSITIONS的最前面，依照ATOMIC_SPECIES中的元素顺序指定 (i)，并赋予无限小的U值。也可以将第3步中计算结果作为新的U值重新迭代计算。
    
    注意该加U方式不支持SOC，而其他加U方式的实现代码并非由原始论文通讯作者Matteo Cococcioni撰写，所以可能并不兼容。
    
    QE example中使用的主要为超软赝势，经测试改用PBE赝势，得到的U值会相差0.2eV左右。
    
2. 进行QE自洽计算 pw.x < qe.in 注意：对于磁性绝缘体材料，因为我们事先并不知道它的基态磁矩大小，所以需要像金属计算那样打开高斯展宽，打开自旋，设置初始磁矩，用比较宽松的收敛条件得到磁矩信息。然后关闭展宽，保持能带数nbnd不变，传入上一步计算得到的磁矩 (舍入到整数位) ，读入上一步的自洽波函数重新自洽计算。以上两步自洽法的输入文件格式参见QE example02。
    
3. 编写hp.in输入文件，使用低密度q点 &inputhp prefix= 'qe', outdir= './', nq1 = 2, nq2 = 2, nq3 =2, conv_thr_chi = 1.0d-8, iverbosity = 2 / 进行hp (hubbard parameter) 模块计算得到U值
    
    ```bash
    hp.x < hp.in
    ```
    
    若材料中有不止一种过渡金属元素，需要用perturb_only_atom先分别计算单个元素，然后用compute_hp汇总得到所有元素U值。以上多步hp计算流程的输入文件格式参见QE example06
    
4. 校验结果
    
    head *.dat
    
    观察到U的取值没有明显问题后，即可扩胞、提高精度重新计算；若第一步参数不恰当，甚至有可能出现负的U值。根据论文结果，取 2x2x2 超胞，4x4x4 K点与q点，相对而言，可以达到较好的收敛精度。超胞大小对U值产生的影响也是最显著的，重新计算得到的U可能和粗算结果有高达2eV的差距。
    
    但需要注意的是，超胞情况下hp计算耗时非常非常惊人，这可能是由于计算复杂度是随原子个数的平方增长的。如果不是特别需要准确的U值，建议不扩胞或者扩胞后继续沿用低密度q点。