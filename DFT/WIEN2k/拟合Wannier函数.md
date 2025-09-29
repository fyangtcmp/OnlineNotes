基于wien2k_19.2版
能带计算完成后，创建子文件夹

```bash
prepare_w2wdir <subdir>
```

切换到 python2.7 环境，运行脚本，注意dmft需要k点为shifted，但这里做wannier需要为unshifted

```bash
init_w2w
```

```bash
x lapw1
x lapwso
x w2w -so -up
x w2w -so -dn
```

```bash
x wannier90 -so
```

这一步计算最后可能会报错“wrongirdist_ws”，是由wannier90-v2.1版导致的，不会影响hr.dat和band.dat的生成。