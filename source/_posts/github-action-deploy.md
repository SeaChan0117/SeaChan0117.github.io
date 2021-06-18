---
title: Github Actions 自动化构建服务
date: 2021-06-18 11:10:04
categories:
- 大前端
- 自动化部署
tags:
- Github Actions
---
**CI/CD** 已经是现代互联网项目的主流，它能帮助我们处理繁琐的打包构建工作，本博客是基于 **Travis CI** 构建的，最近在学习中遇到了 Github Actions，感觉很强大，很好玩，就此记录一下学习过程中的使用。

# Github Actions
GitHub Actions 是 GitHub 于2018年10月推出的持续集成服务。  
官方定义如下，在 GitHub Actions 的仓库中自动化、自定义和执行软件开发工作流程。 您可以发现、创建和共享操作以执行您喜欢的任何作业（包括 CI/CD），并将操作合并到完全自定义的工作流程中。  <!--more-->
总之它的目的，和其他三方平台一样，帮助我们解决构建部署的工作；不一样的是，它无需我们自己编写复杂的打包构建脚本，如果我们需要某项操作时，可以直接下载别人写好的 Action 即可，所有的 Action 构成自己的工作集合，即 Actions。Github 还做了一个[**官方市场**](https://github.com/marketplace?type=actions)，方便我们找到相应的 Action 使用。

# 前置准备
既然是 CI/CD 的操作，前提就是需要有项目和服务器，部署就交给 Github Actions。
## 项目准备
此处我使用了一个基于 Nuxtjs 的 Demo 项目做测试；  
该项目是基于 realworld 模板文件根据原网功能使用 Nuxtjs 完成的；  

* 本次测试项目地址：[https://github.com/SeaChan0117/realworld-nuxtjs](https://github.com/SeaChan0117/realworld-nuxtjs)  
* realworld 模板文件地址：[https://github.com/gothinkster/realworld-starter-kit/blob/master/FRONTEND_INSTRUCTIONS.md](https://github.com/gothinkster/realworld-starter-kit/blob/master/FRONTEND_INSTRUCTIONS.md)  
* 项目使用的后台 API：[https://github.com/gothinkster/realworld/tree/master/api](https://github.com/gothinkster/realworld/tree/master/api)  
* 原项目 Demo 地址：[https://demo.realworld.io/](https://demo.realworld.io)

最后将项目功能开发完。

## 服务器
服务器各自大同小异，我用的是 [ucloud 云主机](https://passport.ucloud.cn/#login)，使用方法参考 [https://juejin.cn/post/6904234342575407111#heading-0](https://juejin.cn/post/6904234342575407111#heading-0)，开启相关端口用于后续部署项目；  
注意，需要在云主机上安装 node 和 pm2，可参考[视频](https://www.bilibili.com/video/BV14v4117712?p=4)；  

## Github Actions 配置
### 生成 Personal access tokens
* 1）Github 登录；
* 2）头像 --> Settings；
* 3）左侧栏 --> Developer settings；
* 4）左侧栏 --> Personal access tokens；
* 5）按钮 Generate new token；
* 6）Note 输入 token 的键值，eg: TOKEN ；
* 7）scopes 选择 repo 即可，用于给 Actions 赋权操作仓库；
* 8）点击最先 Generate token 按钮；
* 9）生成的 token 只会展示一次，自己做好备份，后续会用到；

### 配置项目 Secrets
* 1）进入预要部署项目，eg: https://github.com/SeaChan0117/realworld-nuxtjs；
* 2）点击 Settings；
* 3）左侧栏 --> Secrets；
* 4）点击按钮 New repository secret；
* 5）Name 填写 token 键名，eg：TOKEN_REALWORLD，Value 填写 2.3.1 第 9 步中生成的 token 值，点击 Add secret 按钮；
* 6）重复第 4、5 步，分别生成服务器对应的 secrets，**USERNAME**，**PASSWORD**，**HOST**，**PORT**；  
此处我的设置为，USERNAME --> root，PASSWORD --> *****（保密），HOST  --> 117.50.1.138，PORT --> 8080

### 项目配置文件
1）项目根路径下新增文件 pm2.config.json，内容为：
```json
{
  "apps": [
    {
      "name": "RealWorld",
      "script": "npm",
      "args": "start"
    }
  ]
}
```

2）项目根路径下新建文件夹 .github，下边再建文件夹 workflows，在下边配置 Actions 脚本，此处为 main.yml，即 .github/workflows/main.yml，文件内容如下：  
```yaml
name: Publish And Deploy Demo
on:
  push:
    tags:
      - 'v*' # 提交标签并且内容是 v 开头时，触发自动构建部署

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest # 运行环境
    steps:

    # 下载源码
    - name: Checkout
      uses: actions/checkout@master

    # 打包构建
    - name: Build
      uses: actions/setup-node@master
    - run: npm install
    - run: npm run build
    - run: tar -zcvf release.tgz .nuxt static nuxt.config.js package.json package-lock.json pm2.config.json # 代码构建后，选择后边这些文件及文件夹，压缩为 release.tgz 压缩包

    # 发布 Release
    - name: Create Release
      id: create_release
      uses: actions/create-release@master
      env:
        GITHUB_TOKEN: ${{ secrets.TOKEN_REALWORLD }}
      with:
        tag_name: ${{ github.ref }}
        release_name: Release ${{ github.ref }}
        draft: false
        prerelease: false

    # 上传构建结果到 Release
    - name: Upload Release Asset
      id: upload-release-asset
      uses: actions/upload-release-asset@master
      env:
        GITHUB_TOKEN: ${{ secrets.TOKEN_REALWORLD }}
      with:
        upload_url: ${{ steps.create_release.outputs.upload_url }}
        asset_path: ./release.tgz
        asset_name: release.tgz
        asset_content_type: application/x-tgz

    # 部署到服务器
    - name: Deploy
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        password: ${{ secrets.PASSWORD }}
        port: ${{ secrets.PORT }}
        script: |
          cd /root/realworld-nuxt
          wget https://github.com/SeaChan0117/realworld-nuxtjs/releases/latest/download/release.tgz -O release.tgz
          tar zxvf release.tgz
          npm install --production
          pm2 reload pm2.config.json

```
注意有几个配置项需要修改:  
TOKEN_REALWORLD、HOST、USERNAME、PASSWORD、PORT 对应修改为 2.3.2 中的键名；  
cd 到的目录是项目部署到服务器的目录，得提前保证创建好，这里可以改为自己定义的目录；  
wget 后改为 https://github.com/SeaChan0117/realworld-nuxtjs 改为自己的仓库地址；其他不用修改，可完全复用。   
 
文件地址：  
[https://raw.githubusercontent.com/SeaChan0117/realworld-nuxtjs/main/.github/workflows/main.yml](https://raw.githubusercontent.com/SeaChan0117/realworld-nuxtjs/main/.github/workflows/main.yml)， Ctrl + S 保存为 .yml 文件即可。  

3）将新增的文件提交待远端代码库；
4）给项目打上标签，此处举例为 v0.1.0，推送标签到远端服务;
```
$ git tag v0.1.0

$ git push origin v0.1.0
```
5）此时再到远程仓库刷新后查看，Code 分支下拉选项点开，Branches 旁边的 Tag 打开后发现多了 v1.0.1 的标签；
6）打开仓库的 Actions，发现有名为 Publish And Deploy Demo 的工作流，右侧正在构建，进入可查看构建过程及一系列 Actions 的结果。
7）部署任务完全成功后，根据 HOST 和 PORT 访问项目，浏览器直接访问 [117.50.1.138:8080](http://117.50.1.138:8080/)，Bingo！

以上就是通过提交 GitHub Actions 进行自动化构建部署的过程。

# 相关链接
GitHub 官方的 action：[https://github.com/actions](https://github.com/actions)  
GitHub 官方市场中的 action：[https://github.com/marketplace?type=actions](https://github.com/marketplace?type=actions)  
第三方收集的有用的 action：[https://github.com/sdras/awesome-actions](https://github.com/sdras/awesome-actions)  

# 总结
好玩！强大！

