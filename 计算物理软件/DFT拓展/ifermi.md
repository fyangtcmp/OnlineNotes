## 安装

```bash
conda create --name ifermi
conda activate ifermi
conda install -c conda-forge pymatgen boltztrap2 pyfftw=0.12.0
pip install ifermi
```
必须安装旧版的 pyfftw
## 使用

我们知道，VASP 会根据晶体对称性，将 KPOINTS 中定义的均匀 k 点网格进行化简，变成 IBZKPT 中带有不同权重的一系列不可约 k 点。
因为费米速度是可以直接从能带计算的，所以 ifermi 在将不可约 k 点上的数据还原成均匀网格的过程中会自动得到速度。
但自旋投影信息是只能由 VASP 给出的，它目前无法从不可约 k 点还原至均匀网格，因此 `--property spin` 需要基于 `ISYM=0` 的计算

必须是自洽计算的数据，不能读取非自洽计算

```
ifermi plot --mu 0 --slice 0 0 1 0 --property spin --hide-cell \
            --hide-labels --projection-axis 0 0 1 --property-colormap RdBu \
            --vector-property --vector-colormap RdBu --vnorm 5 --vector-spacing 0.02 --output fsurf-slice.png
```

 --scale 20

```
ifermi plot --mu 0 --slice 0 1 0 0 --property spin --hide-cell \
            --hide-labels --projection-axis 0 1 0 --property-colormap RdBu \
            --vector-property --vector-colormap RdBu --vnorm 5 --vector-spacing 0.025 \
            --output fsurf-slice.png
```




```bash
ifermi plot --projection-axis 0 0 1 --property spin \
            --plot-index 169 --vnorm 3 --vector-spacing 0.02 \
            --vector-property \
            --property-colormap RdBu --vector-colormap RdBu \
            --hide-surface --hide-labels \
            --output fsurf.html
```

```
--plot-index 169 \
--vnorm 3 --vector-spacing 0.025 \
```