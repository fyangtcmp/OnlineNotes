[ccao/WannSymm: Symmetry analysis and symmetrize in Wannier orbitals (github.com)](https://github.com/ccao/WannSymm)
POSCAR 中的原子坐标不能有负值
wannier90 默认输出的 `hr.dat` 文件中跃迁项的有效位数只有小数点后 7 位，这同样也会削弱对称性的数值精度。使用 WannSymm 对称化以后有效位数将提高至小数点后 16 位。