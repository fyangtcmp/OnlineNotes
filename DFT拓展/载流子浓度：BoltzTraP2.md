## 安装
```bash
conda install -c conda-forge boltztrap2 pyfftw=0.12.0
```
必须像这样指定安装旧版pyfftw

## 计算
先做一个高密度 k-mesh 的 vasp 自洽计算，不需要 WAVECAR 和 CHGCAR。然后输入
```
btp2 -vv interpolate -m 5 ./
```
`-vv` 代表显示较多的日志信息，`-m` 设置每个 k 点周围插值点的数目
生成 interpolation.bt2 文件，然后输入
```
btp2 -vv integrate interpolation.bt2 100:105:2
```
代表以 2K 的间隔，从 100K 至 105K 计算电导率数据（不包括右边界）。因为涉及到费米分布导数的积分，所以在低温区展宽非常小的情况下计算耗时会非常长（几个小时），高温区例如 50K 以上计算会很快（十几分钟）。并行参数此处不起效

## 画图
```
btp2 plot -u -c  '["xx", "yy"]' -s 10 interpolation.btj sigma
btp2 plot -T -c  '["xx", "yy"]' -s 1000 interpolation.btj sigma
```
 `-u` 将能量设置为 x 轴，不同温度对应不同曲线； `-T` 将温度设置为 y 轴，不同费米面对应不同曲线
 `-c` 选择画出哪几个分量，比如这里是 xx 和 yy 分量
  `-s` 设置每隔多少个点画一条曲线。因为之前我们温度计算时输入 `100:105:2`，相当于只有 3 个不同的温度点，所以以能量为横坐标时可以设为默认值 1。但能量采样点的数目非常多，所以以温度为横坐标时要设的非常大
  `sigma` 选择画出电导率，其他支持的物理量可通过
```
btp2 plot -h
```
  来查询

画载流子浓度
```
btp2 plot -u -s 10 interpolation.btj n
```