
原始文献

[https://doi.org/10.1107/S160057672001554X](https://doi.org/10.1107/S160057672001554X "https://doi.org/10.1107/S160057672001554X")


源码

[asche1/PyCrystalField: Code to calculate the crystal field Hamiltonian of magnetic ions. (github.com)](https://github.com/asche1/PyCrystalField "asche1/PyCrystalField: Code to calculate the crystal field Hamiltonian of magnetic ions. (github.com)")

## 对称性

cif文件的对称性，对能量本征值没有明显影响，但无对称性的本征矢量容易出现虚部
## SOC强度

soc耦合为正，基态与第一激发态能量差距较小；

soc耦合为负，基态与第一激发态能量差距较大；

能量差与SOC耦合强度近似成正比
## 价态设置

需要为中央的金属元素和晶体场中所有非金属元素同时设置价态，如果只设置金属价态，会被程序忽略。

标号写在\_atom\_site\_label

价态写在\_atom\_site\_type\_symbol