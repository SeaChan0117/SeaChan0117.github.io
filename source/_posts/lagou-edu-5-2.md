---
title: NodeJS + Redis 实现漂流瓶接口
date: 2021-10-19 22:02:54
categories:
- 大前端
- node
tags:
- 大前端
- node
cover: https://tse1-mm.cn.bing.net/th/id/OIP-C.Wb-Xm0eazG3N8h6f5mgT3wHaD4?pid=ImgDet&rs=1
---

NodeJS + Redis 实现漂流瓶接口
<!--more-->

## 界面：http://117.50.1.138/

## api:
### 扔瓶子：post http://117.50.1.138:3000
* time：漂流瓶扔出的时间戳，默认时设置为 Date.now()，后台生成，前台不用传；
* owner：漂流瓶主人；
* type：漂流瓶类型，为 male 或 female 之一；
* content：漂流瓶内容；

### 捞瓶子：get  http://117.50.1.138:3000?user=xxx[&type=xxx]
* user：捡漂流瓶的人的用户名或用户id；
* type：漂流瓶类型，这里我们设置三种类型：all 代表全部，male 代表男性，female 代表女性，默认时为 all；
