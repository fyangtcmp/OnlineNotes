## 用户管理

新建用户，指定shell，建立对应的家目录（ubuntu）
```bash
adduser <name>
```

删除用户及家目录
```bash
userdel -r <name>
```

## 戴尔服务器运行日志

[PowerEdge — 如何查看/清除系统事件日志 (SEL) | Dell 中国](<https://www.dell.com/support/kbdoc/zh-cn/000132405/poweredge-如何-查看-清除-系统-事件-日志-sel#3.Viewing System Event Log using RACADM> "PowerEdge — 如何查看/清除系统事件日志 (SEL) | Dell 中国")

<https://www.linuxidc.com/Linux/2019-08/159730.htm>

## 解除服务器内存限制

```bash
vi /etc/security/limits.conf
```

```
* soft memlock unlimited
* hard memlock unlimited
* soft stack unlimited
* hard stack unlimited
* hard nofile 1000000
* soft nofile 1000000
* hard nproc 2000000
* soft nproc 2000000
```

## 并行控制
使用 intel 编译的程序，需要配合 intel 的 mpirun 才不容易出问题。但如果你启用了 conda 环境的话，里面有可能也带着一个 python 版的 mpirun，这时候如果你在 bashrc 里是先 source intel 的环境，后激活 conda，就会导致 python 的 mpirun 覆盖掉 intel 的 mpirun