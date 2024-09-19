[Vampire (york.ac.uk)](https://vampire.york.ac.uk/index.html#introduction)
## 单位制

大多数计算物理软件，如 VASP、QE 等，在输入文件中设定参数时，只有一个单独的数值。譬如
```
EDIFF = 1e-6
```
但它这里其实暗藏了一个能量量纲的单位 eV，而且这个单位制是不能改动的。

Vampire 则对输入参数的单位制提供了丰富的支持，但由于这一语法没有在官方资料中明确提及，很容易导致初学者的迷惑，以长度量纲的参数为例，默认情况下
```
dimensions:system-size-x = 2
```
以埃为单位，这等价于
```
dimensions:system-size-x = 2 !A
```
如果采用
```
dimensions:system-size-x = 2 !nm
```
Vampire 可以自动将其识别为 2 纳米。这里感叹号是必须的，不可略去。所有支持的单位制可在 vampire-6.0/src/utility/units.cpp 这一源代码文件中查阅。

## 比热容

输出
比热单位制

VAMPIRE should output a mean-specific-heat or a material-mean-specific-heat in kB/spin.  

The more challenging part might be how to get spin to kg.
  
Maybe you could assume 1 spin equals 1 atom.  If so, from the geometry of the cell that you've specified in the VAMPIRE simulation, you likely could determine the atomic density [5] in units of atoms/cell_volume.  

From that, I think could get the volumetric heat capacity.  Then, according to [6], the volumetric heat capacity can multiplied by the material or mass density to express the heat capacity per unit of mass (i.e., units of JK-1kg-1).
## 文件格式
vampire.materials 文件中，material 和 unit-cell-category 的计数都是从 1 开始的。但 vampire.UCF 文件中，定义原子位置时
```
atom_id cx cy cz [mat_id cat_id hcat_id]
```
mat_id 的计数是从 0 开始的，cat_id 的计数依然是从 1 开始的

## 磁滞回线
有限温度计算磁滞回线是非常困难的，因为数值模拟的时间尺度基本只能维持在微秒量级，但实验上测量得到磁滞回线往往需要持续数分钟的加磁或退磁过程，参见[Reduce noise in hysteresis loops (google.com)](https://groups.google.com/g/vampire-users/c/3FQwxSDv65I)

## 居里温度/奈尔温度
计算居里温度，monte-carlo 算法和 llg-heun 算法几乎没有区别，llg-heun 算法有很小的速度优势，monte-carlo 给出的比热曲线更加光滑。

## 可视化
```
config:atoms-output-rate = 1
sim:time-steps-increment=
```
是对于单个温度而言的