## 字体安装
[Ubuntu安装字体教程（用户级/系统级） | Patrick's Blog (patzer0.com)](https://patzer0.com/archives/ubuntu-install-font-for-user-or-system)

## 版本选择
Ubuntu 有桌面版、服务器版两个版本。
因为服务器版没有图形化界面和火狐，在校园网环境需要 web 页面认证联网的环境下不方便，所以我们先安装桌面版，然后执行
```bash
sudo apt-get install  linux-image-server linux-headers-server linux-server
```

使用服务器内核，安装成功以后重新启动