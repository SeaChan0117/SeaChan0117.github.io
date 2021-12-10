---
title: 鸿蒙OS JS UI
date: 2021-11-02 20:24:53
categories:
- 大前端
- 鸿蒙
tags:
- 鸿蒙
cover: https://tse2-mm.cn.bing.net/th/id/OIP-C.RM25FYNbXlYsmNDuLdEhwwHaD8?pid=ImgDet&rs=1
---

# 鸿蒙OS JS UI 框架

## 什么是鸿蒙
* 鸿蒙（HarmonyOS）是一款由华为开发的，面向全场景的分布式操作系统，其开源项目为 OpenHarmony。
* 超级小程序
    * H5 -> 小程序（可使用移动端功能，离不开微信等超集 APP） -> 超级小程序（脱离了 APP 的限制，依旧能使用移动端功能，且可跨设备使用移动端功能，eg: 手机端操作其他设备）
    
* 可剪裁的系统
    * 128 KB - 128 MB - 4 GB （根据硬件的大小或强弱，选择性地运行整个操作系统的某些部分）
    
* 模改的通讯协议
    * 类似普通话统一方言，鸿蒙成为 IOT(Internet Of Things) 互联互通的标准语言
    
## 发展历史
* 2012年，华为开始规划自有操作系统 “鸿蒙”
* 2019年8月9日，华为发布鸿蒙 1.0
* 2020年9月10日，华为发布鸿蒙 2.0
* 2021年6月2日，鸿蒙正式商用
    * 华为正式发布 HarmonyOS 2.0 及多款搭载 HarmonyOS 2.0 的新产品
    
## 相关网站
* 鸿蒙官网：[https://www.harmonyos.com/](https://www.harmonyos.com/)
* 鸿蒙系统开发者：[https://developer.harmonyos.com/](https://developer.harmonyos.com/)
* 华为开发者：[https://developer.huawei.com/cn/](https://developer.huawei.com/cn/)
* 在线体验：[https://playground.harmonyos.com/](https://playground.harmonyos.com/)
* Gitee：[https://gitee.com/openharmony](https://gitee.com/openharmony)
* JS API：[https://developer.harmonyos.com/cn/docs/documentation/doc-references/js-apis-overview-0000001056361791](https://developer.harmonyos.com/cn/docs/documentation/doc-references/js-apis-overview-0000001056361791)

## 开发前知识储备
* 熟悉前端技术栈（HTML、CSS、JS）
    * 鸿蒙开发不是浏览器环境，里面的关键字，都是仿 Web 端的（eg：鸿蒙中的 div 是自己封装的，不是 Web 端的 div）
    
* 熟悉微信小程序
    * 页面结构、API、配置方式等都是参照小程序
    
* 熟悉 Vue
    * 数据绑定、自定义组件、全局变量等都是参照 Vue 2
    
* 有安卓开发经验更好
    
# 系统架构
鸿蒙系统架构⼀共分四层：应⽤层、框架层、系统服务层、内核层。
![系统架构](系统架构.png)
## 架构
* 应用层
    * 鸿蒙的应用由一个或多个 FA(Feature Ability) 或 PA(Particle Ability) 组成
    * FA 有 UI 界面，提供与用户交互的能力，而 PA 无 UI 界面，提供后台运行任务的能力


因为某些原因暂时没时间学习鸿蒙啦，后续学习会继续更新
