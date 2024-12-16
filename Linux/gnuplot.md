在. gnu 文件开头添加以下指令，以保存绘图文件到本地

```纯文本
set term post eps color
set output 'band.eps'
```

服务器可能没有 png 库，所以设置为 eps 格式。