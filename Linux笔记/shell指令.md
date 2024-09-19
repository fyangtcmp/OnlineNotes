## 统计文件个数

统计当前目录下文件的个数（不包括目录）
```bash
ls -l | grep "^-" | wc -l
```

统计当前目录下文件的个数（包括子目录）
```bash
ls -lR | grep "^-" | wc -l
```

统计当前目录下文件夹的个数
```bash
ls -lR | grep "^d" | wc -l
```
## 解除服务器堆栈上限

```bash
ulimit -s unlimited
```

一般用于解决vasp报错：forrtl: severe（174）：SIGSEGV, segmentation fault occurred
## 查看系统硬盘占用

查看文件夹大小
```bash
du -sh <name>
```

查看硬盘挂载各分区的使用量
```bash
df -h
```

注意它们给出的大小并不总是等价的，原因参见

[使用du与df命令查看磁盘容量不一致 (aliyun.com)](https://help.aliyun.com/document_detail/96228.html "使用du与df命令查看磁盘容量不一致 (aliyun.com)")
## 命令行模式匹配、查询、修改、删除

使用sed指令，例如删除\<target\_text>所在的一整行

```bash
sed -i '/<target_text>/d' <filename>
```

sed -i代表直接操纵目标文件，如果不加-i的话，sed会将结果打印出来，保持目标文件内容不变。如果\<target\_text>出现过不止一次，那么每次出现的一整行都会被删除。

如果只想对第一次出现的文本进行操作，在引号内字符串的前方添加"0,"，例如

```bash
sed -i '0,/<target_text>/d' <filename>
```

匹配\<target\_text>所在的行，并将那一整行的内容替换为\<new\_line>

```bash
sed -i "/<target_text>/c\<new_line>" <filename>
```

匹配\<target\_text>所在的行，仅将该部分文本替换为\<new\_text>

```bash
sed -i '/<target_text>/{ s/<target_text>/<new_text>/;}' <filename>
```
 
## 程序后台运行

[Linux 前台任务转后台运行(会话中断不停止) - 简书 (jianshu.com)]( https://www.jianshu.com/p/89edba7c6805 "Linux 前台任务转后台运行(会话中断不停止) - 简书 (jianshu.com)")
