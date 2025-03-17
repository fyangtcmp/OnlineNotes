## 字体安装
[Ubuntu安装字体教程（用户级/系统级） | Patrick's Blog (patzer0.com)](https://patzer0.com/archives/ubuntu-install-font-for-user-or-system)

## 版本选择
Ubuntu 有桌面版、服务器版两个版本。
因为服务器版没有图形化界面和火狐，在校园网环境需要 web 页面认证联网的环境下不方便，所以我们先安装桌面版，然后执行
```bash
sudo apt-get install  linux-image-server linux-headers-server linux-server
```

使用服务器内核，安装成功以后重新启动

## 逻辑系统盘扩容
在我们安装ubuntu时，如果选择的是自动分区，就会按照逻辑卷的形式来分区，并且只分配 100G 其余的并不会被分配。
如果在执行 apt 包管理时出现写入硬盘错误，很有可能就是系统盘的逻辑卷空间不够导致的
https://www.cnblogs.com/guangdelw/p/17822292.html