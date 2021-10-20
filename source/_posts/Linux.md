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

> **用于记录平时开发常用的远程操作命令，不定期更新；**

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
