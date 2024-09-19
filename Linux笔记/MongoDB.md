## 安装启动

启动

```bash
numactl --interleave=all mongod --config /DATA/mongoDB/etc/mongodb.conf
```

关闭

```bash
mongod --shutdown -config /DATA/mongoDB/etc/mongodb.conf
```

## 用户管理

网上教程一般是从安装包bin文件夹内直接运行./mongo指令来管理数据库，但最新版MongoDB已经不再附带安装mongo shell，需要单独下载，运行指令也变为mongosh

第一次启动后，创建超级管理员，在命令行界面下输入

```bash
use admin
```

以切换到admin数据库，输入

```bash
db.createUser(
{ user: "admin",
pwd: "I_am_not_root",
roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
}
)

```

注意：MongoDB的admin账户不同于传统意义上的超级用户，role的前缀"userAdmin"，代表只有用户的管理权限，并不能直接读写所有数据库

创建具有读写所有数据库权限的用户，继续在admin数据库中操作（修改用户很麻烦，网上教程混乱，最好这一步就建好）

```bash
db.createUser({
user:"XXX",
pwd:"mmmmmm",
roles:[
{role:"readWriteAnyDatabase",db:"admin"},
]
})
```

用户创建完毕后，可使用auth指令在mongosh内部登录
 
```bash
db.auth('XXX','mmmmmm')
```

或者在运行mongosh时就输入

```bash
mongosh -u XXX -p mmmmmm
```

注意以上过程中密码均会明文出现。

最后提醒，用户组是附属于数据库的，一个用户的完整索引为dbname.username，所以在创建时要确认好目前命令行所在的数据库。
## 数据库管理

如果数据库中没有任何内容的话，只会隐式创建，退出后是无法访问到的
### Python接口

[Python操作MongoDB看这一篇就够了 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1151814 "Python操作MongoDB看这一篇就够了 - 腾讯云开发者社区-腾讯云 (tencent.com)")

### MATLAB接口

[Create MongoDB C++ interface connection - MATLAB mongoc - MathWorks 中国](https://ww2.mathworks.cn/help/database/ug/mongoc.html#responsive_offcanvas "Create MongoDB C++ interface connection - MATLAB mongoc - MathWorks 中国")