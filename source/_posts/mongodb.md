---
title: mongodb
date: 2021-10-28 09:34:08
categories:
- 大前端
tags:
- MongoDB
cover: https://tse1-mm.cn.bing.net/th/id/OIP-C.yyxuvK8jsaQIYEnh2cxeagHaD_?pid=ImgDet&rs=1
---

# Linux 下安装 MongoDB
## 下载安装包
官方下载：[https://www.mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)
![下载](下载.png)

* 可以下载到本地上传至服务器；
* 也可以 `Copy Link` 后到服务器命令下载

```sh
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel80-5.0.3.tgz
```

## 安装
### 解压安装包
```sh
tar -zxvf mongodb-linux-x86_64-rhel80-5.0.3.tgz
```
### 将解压后的目录移动到 `/usr/local` 目录下，并改名为 `mongodb`
```sh
mv mongodb-linux-x86_64-rhel80-5.0.3 /usr/local/mongodb
```
### 进入 `mongodb` 目录，并创建文件夹 `data`，在 `data` 文件夹下再创建 `db` 文件夹（用于存放数据库数据）和 `log` 文件夹（存放 `mongo` 日志），然后为其设置可读写权限
```sh
# 进入目录
cd /usr/local/mongodb/

# 创建三个文件夹
mkdir data data/db data/log

# 设置可读写权限
sudo chmod 666 data/db data/log/
```
### 配置文件  
> 在 `mongodb` 目录下新建配置文件 `mongodb.conf`（可选，但建议配置），打开文件输入以下内容

```sh
# 数据库数据存放目录
dbpath=/usr/local/mongodb/data/db
# 日志文件存放目录
logpath=/usr/local/mongodb/data/log/mongodb.log
# 日志追加方式
logappend=true
# 端口
port=27017
# 是否认证
auth=true
# 以守护进程方式在后台运行
fork=true
# 远程连接要指定ip，否则无法连接；0.0.0.0代表不限制ip访问
bind_ip=0.0.0.0
```
### 配置环境变量
```sh
# 进入编辑
vim /etc/profile

# 在最后添加
export MONGODB_HOME=/usr/local/mongodb
export PATH=$PATH:$MONGODB_HOME/bin

# 重启系统配置
source /etc/profile
```
### 启动
```sh
mongod -f /usr/local/mongodb/mongodb.conf
```
![启动成功](启动成功.png)

### 验证
使用安装目录下 `bin` 目录的 `mongo` 客户端命令连接和访问 `MongoDB`，默认会链接到 `test` 数据库
```sh
[root@10-9-66-224 mongodb]# mongo
MongoDB shell version v5.0.3
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("5bae68b0-2b00-4b0c-b154-4a5e15f09c86") }
MongoDB server version: 5.0.3
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
We recommend you begin using "mongosh".
For installation instructions, see
https://docs.mongodb.com/mongodb-shell/install/
================
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
        https://docs.mongodb.com/
Questions? Try the MongoDB Developer Community Forums
        https://community.mongodb.com
> db
test
>
```

## 其他配置
### 开放端口
```sh
# 开放端口
firewall-cmd --zone=public --add-port=27017/tcp --permanent
# 重新加载规则. 使之立刻生效
firewall-cmd --reload
# 查看开放的端口，验证是否成功
firewall-cmd --zone=public --list-ports
```
### 检查服务状态
```sh
# 查看 mongodb 进程状态
ps aux | grep mongo

# 检查端口是否启动
netstat -lanp | grep 27017
```
### 停止服务
```sh
# 通过进程 ID 杀死
kill -9 PID

# 通过 mongod 命令关闭 mongodb 服务
mongod -f /usr/local/mongodb/mongodb.conf --shutdown
```
### 开机自启动
```sh
# 新建配置文件
vim /lib/systemd/system/mongodb.service
```
配置文件中填写以下内容
```sh
[Unit]
    Description=mongodb
    After=network.target remote-fs.target nss-lookup.target
[Service]
    Type=forking
    ExecStart=/usr/local/mongodb/bin/mongod -f /usr/local/mongodb/mongodb.conf
    ExecReload=/bin/kill -s HUP $MAINPID
    ExecStop=/usr/local/mongodb/bin/mongod -f /usr/local/mongodb/mongodb.conf --shutdown
    PrivateTmp=true
[Install]
    WantedBy=multi-user.target
```
依次执行以下四个命令，使之生效
```sh
# 启动 MongoDB
systemctl start mongodb.service

# 查看服务状态
systemctl status mongodb.service

# 开机自启动
systemctl enable mongodb.service

# 修改 mongodb.service文件时，重新加载文件
systemctl daemon-reload
```

### 启动重启停止服务
```sh
systemctl start mongodb.service
systemctl restart mongodb.service
systemctl stop mongodb.service
```

## 角色及用户名配置
> 配置文件中，指定了 `auth=true`，即开启了认证，所以链接后需要认证才能执行操作。默认情况下，`MongoDB` 是没有管理员账户的，所以我们需要在 `admin` 数据库中使用 `db.createUser()` 命令添加管理员帐号或其他角色。

### 内置角色
> * 数据库用户角色：`read`、`readWrite`
> * 数据库管理角色：`dbAdmin`、`dbOwner`、`userAdmin`
> * 集群管理角色：`clusterAdmin`、`clusterManager`、`clusterMonitor`、`hostManager`
> * 备份恢复角色：`backup`、`restore`
> * 所有数据库角色：`readAnyDatabase`、`readWriteAnyDatabase`、`userAdminAnyDatabase`、`dbAdminAnyDatabase`
> * 超级用户角色：`root`
> * 内部角色：`__system`

### 创建管理员账号
切换到 admin 数据库，使用以下命令创建管理账号，拥有操作所有数据库权限
```sh
# 连接进入 mongodb 命令下
mongo

# 切换到 admin 数据库
use admin

# 创建用户
db.createUser({user:"admin",pwd:"123456",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})

```
![创建用户](创建用户.png)

### 验证
使用 `mongo` 命令连接上之后，如果不进行 `db.auth("用户名","密码")` 进行用户验证的话，是执行不了任务命令的，只有通过认证才可以。注意，每一个用户都需要在创建这个用户的认证库下进行认证。
```sh
mongo

use admin

db.auth("admin","123456")

show tables
```
![用户验证](用户验证.png)

### 可视化工具连接
我使用的是 *`Navicat 15 for MongoDB`*
![Navicat连接](Navicat连接.png)

## 参考 [https://blog.csdn.net/chenlixiao007/article/details/110206062](https://blog.csdn.net/chenlixiao007/article/details/110206062)
