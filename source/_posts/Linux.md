---
title: 远程服务器 Linux 常用操作
date: 2021-10-20 17:08:30
categories:
- 大前端
tags:
- linux
cover: https://tse1-mm.cn.bing.net/th/id/R-C.91582b5b7f60eac40848e7a8dafb63bd?rik=%2fsYtXy2uanfBuw&riu=http%3a%2f%2fwww.linuxidc.com%2fupload%2f2017_07%2f170701202019191.jpg&ehk=yvJ%2bf76veYuzTFzjWYASaHpZ7n%2b88DcBYD7otSc3Wpw%3d&risl=&pid=ImgRaw&r=0
---

# About

> **本文用于记录平时开发常用的远程操作命令，不定期更新；**

# 安装 node / pm2 / 配置相关路径
## 脚本安装

```shell
# 命令1 && 命令2   命令1执行完成后，再执行命令2
nver='v14.16.1'                 #定义版本变量 nver
ndir="node-${nver}-linux-x64"   #定义目录变量 ndir 
nfile="${ndir}.tar.xz"          #定义压缩文件名变量 nfile

cd /usr/local                   #切换目录
wget https://nodejs.org/dist/$nver/$nfile  $下载文件
tar xvf $nfile    #文件拆包解压
mv $ndir nodejs   #对目录重命名
rm -rf $nfile     #删除压缩包文件

cd nodejs/bin     #进入目录

# 获取真实路径, 软链接到 /usr/bin 中, 使命令全局可用. -f为强制创建，会覆盖
ln -sf `readlink -f node` /usr/bin/node
ln -sf `readlink -f npm`  /usr/bin/npm
ln -sf `readlink -f npx`  /usr/bin/npx


# 配置镜像
npm config set registry http://registry.npm.taobao.org
# 全局安装pm2
npm i pm2 -g   
# 建立软链接. 使pm2全局使用
ln -sf `readlink -f pm2`  /usr/bin/pm2
# 返回目录
cd
```

## 通过 nvm 安装 ( github 不稳定，暂时不用这种方式 )

```shell
# 1.安装 nvm
cd
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash

# 2.断开+重新连接 或 执行下面命令使 nvm 生效
  source ~/.bashrc # bash
# source ~/.zshrc  # zsh  MacOS ?

# 3.安装 node 稳定版
nvm install --lts

# 4.安装 pm2 到全局
npm i pm2 -g

# 5.建立软链接
nver='v14.16.1' && cd /root/.nvm/versions/node/$nver/bin &&  ln -sf `readlink -f node` /usr/bin/node && ln -sf `readlink -f npm`  /usr/bin/npm && ln -sf `readlink -f npx`  /usr/bin/npx && ln -sf `readlink -f pm2`  /usr/bin/pm2 && cd
```

# 内部防火墙

**( 建议不要开启内部防火墙 )**

```shell
# 关闭内部防火墙
systemctl stop firewalld      # 临时关
systemctl disable firewalld   # 永久关. 重启以后也是关着的

# 开启内部防火墙
#systemctl start firewalld
#查看防火墙状态
#systemctl status firewalld.service

# 添加开放端口规则
#firewall-cmd --zone=public --add-port=22/tcp   --permanent
#firewall-cmd --zone=public --add-port=80/tcp   --permanent
#firewall-cmd --zone=public --add-port=443/tcp  --permanent
#firewall-cmd --zone=public --add-port=1337/tcp --permanent
#firewall-cmd --zone=public --add-port=3000/tcp --permanent
#firewall-cmd --zone=public --add-port=3306/tcp --permanent
#firewall-cmd --zone=public --add-port=8080/tcp --permanent

# 重新加载规则. 使之立刻生效.
#firewall-cmd --reload
```

# 安装配置 GIT
## 安装

```shell
yum -y install git
```

## SSH 配置 GIT

1. 进入当前用户目录，输入以下命令，连续回车 3 次；

    * key生成成功后用户主目录生成.ssh文件夹；
    * id_rsa是私钥，id_rsa.pub是公钥；

   ```shell
   ssh-keygen -t rsa -C "自己的github账号"
   ```

2. 查看 `id_rsa.pub` 公钥内容，秘钥信息以 `ssh-rsa` 开始，邮箱结束；

3. 登录自己的 `github` ，找到 `Settings=>SSH and GPG keys` 设置；

4. 点击 `New SSH key` ，`Title` 输入框随意，`Key` 输入框填入步骤 *2* 公钥的内容；

5. 服务器命令窗口输入以下命令测试是否成功；

   ```shell
   ssh -T git@github.com
   ```

6. 成功后即可 `clone` 代码；

# PM2 常用命令

```shell
npm run dev  等同于 pm2 start  npm -- run dev
npm start  等同于 pm2 start npm -- start 

# 命名进程名
pm2 start npm --name test -- run dev
pm2 start npm --name test -- start 

# 查看
pm2 list
# 重载进程
pm2 reload <id>
```

# 安装 MySQL

## 命令下载 MySQL，也可离线下载后上传到服务器

```sh
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz
```

## 解压

```sh
tar -xvf mysql-8.0.25-linux-glibc2.12-x86_64.tar.xz
```



## 移动解压后的文件夹到指定目录，并修改文件夹名为 `mysql-8.0.25`

此处目录以 `/data01/mysql/` 为例，实际根据自己需求操作，保证后续相关路径和该目录匹配即可；

```sh
mv mysql-8.0.25-linux-glibc2.12-x86_64 /data01/mysql/mysql-8.0.25
```



## 创建数据目录

```sh
# 1.创建文件夹
mkdir -p /data/mysqldata/
#2 创建数据库用户 后边文件配置及初始化会用到, 如果你自己有其他用户也可以不创建新的
	#2.1创建用户组
	groupadd mysql
	#2.2创建用户
	useradd -r -g mysql mysql
#赋权限
2. chown mysql:mysql -R /data/mysqldata #chown 用户名:用户组 -R /data/mysqldata
3. chmod 750 /data/mysqldata/ -R
```

## 环境配置

```sh
vim /etc/profile 
#如果你的系统不支持vim命令 使用下边这个
vi /etc/profile
#编辑,在文档最后一行 添加下边代码
export PATH=$PATH:你的MySQL解压路径/mysql-8.0.25/bin:你的MySQL解压路径/mysql-8.0.25/lib
```
![配置profile](配置profile.png)
## 编辑 `my.cnf`

* 打开文件

```sh
vim /etc/my.cnf #或者 vi /etc/my.cnf 
```

* 按insert 进入编辑模式 添加以下脚本

```sh
[mysql]
# 客户端默认字符集
default-character-set=utf8mb4
[client]
port=3306
socket=/var/lib/mysql/mysql.sock
[mysqld]
port=3306
server-id=3306
user=mysql
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# 设置mysql的安装目录
basedir=/data01/mysql/mysql-8.0.25 #你自己的安装路径
# 设置mysql数据库的数据的存放目录
datadir=/data/mysqldata/mysql  #你自己创建的数据库文件存放路径
log-bin=/data/mysqldata/mysql/mysql-bin
innodb_data_home_dir=/data/mysqldata/mysql
innodb_log_group_home_dir=/data/mysqldata/mysql
character-set-server=utf8mb4
lower_case_table_names=1
autocommit=1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld_safe]
#设置mysql数据库的日志及进程数据的存放目录
log-error=/data/mysqldata/mysql/mysql.log
pid-file=/data/mysqldata/mysql/mysql.pid
```

## 初始化 `MySQL`

```sh
cd /data01/mysql/mysql-8.0.25/bin/
./mysqld --defaults-file=/etc/my.cnf --basedir=/data01/mysql/mysql-8.0.25/ --datadir=/data/mysqldata/mysql --user=mysql --initialize

# 参数详解
# --defaults-file=/etc/my.cnf 指定配置文件（一定要放在最前面，至少 --initialize 前面）
# --user=mysql 指定用户（很关键）
# --basedir=/data01/mysql/mysql-8.0.25/ 指定安装目录
# --datadir=/data/mysqldata/mysql/ 指定初始化数据目录
```

初始化数据库后，会给一个临时密码，请保存到本地，第一次登录数据库会用到；(dj7=M4ajHok1)

```sh
[root@10-9-66-224 bin]# ./mysqld --defaults-file=/etc/my.cnf --basedir=/data01/mysql/mysql-8.0.25/ --datadir=/data/mysqldata/mysql --user=mysql --initialize
2021-10-26T01:51:48.835319Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2021-10-26T01:51:48.835415Z 0 [System] [MY-013169] [Server] /data01/mysql/mysql-8.0.25/bin/mysqld (mysqld 8.0.25) initializing of server in progress as process 127026
2021-10-26T01:51:48.903706Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2021-10-26T01:51:52.076838Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2021-10-26T01:51:53.498736Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: dj7=M4ajHok1
```
![初始化成功](初始化成功.png)
## 启动 `MySQL`

```sh
# 1.复制 mysql.server 文件
cp /data01/mysql/mysql-8.0.25/support-files/mysql.server /etc/init.d/mysql
cp /data01/mysql/mysql-8.0.25/support-files/mysql.server /etc/init.d/mysqld
# 2.赋予权限
chown 777 /etc/my.cnf
chmod +x /etc/init.d/mysql
chmod +x /etc/init.d/mysqld
# 3.检查一下/var/lib/mysql是否存在，否进行创建
mkdir /var/lib/mysql #目录和my.cnf保持一致
# 4.赋予权限
chown -R mysql:mysql /var/lib/mysql/ #目录和my.cnf保持一致
# 5.启动数据库
service mysql start #或者 systemctl mysql start 
```
![启动成功](启动成功.png)

## `MySQL` 数据库设置

### 修改初始密码

```sh
# 登录
mysql -u root -p
# 输入上述保存的初始化密码（dj7=M4ajHok1）
# 修改root密码 修改root用户只能本地连接
ALTER USER 'root'@'localhost' IDENTIFIED with mysql_native_password BY '新密码';
# 刷新权限
flush privileges;
```
![登录](登录.png)
![修改root密码](修改root密码.png)
### 创建用户

```sh
# 创建用户任意远程访问
CREATE USER 'seachan'@'%'; 
# 修改密码
ALTER USER 'seachan'@'%' identified with mysql_native_password by '新密码'; 
```
![创建远程连接新用户及修改密码](创建远程连接新用户及修改密码.png)
### 创建数据库

```sh
create database mag default character set utf8mb4 collate utf8mb4_unicode_ci;
# create database 数据库名 default character set utf8mb4 collate utf8mb4_unicode_ci;
```

### 授权

```sh
# 将 mag 库的所有权限赋予 seachan 用户
grant all privileges on mag.* to "seachan"@"%";
# 刷新权限
flush privileges; 
```

### `Navicat` 远程测试
![Navicat连接测试](Navicat连接测试.png)
![Navicat打开](Navicat打开.png)
## 扩展-开机自启动

```sh
#1.查看是否有mysql服务
chkconfig --list
#2.进入mysql软件目录，复制mysql.server文件到 /etc/rc.d/init.d目录下
cp /data01/mysql/mysql-8.0.25/support-files/mysql.server /etc/rc.d/init.d/mysql
#3.给/etc/rc.d/init.d/mysql赋权可执行权限
chmod +x /etc/rc.d/init.d/mysql
#4.添加mysql服务
chkconfig --add mysql
#5.使mysql服务开机自启
chkconfig --level 345 mysql on
#6.查看MySQL服务 ,重启服务器，测试是否成功。
chkconfig --list
```

