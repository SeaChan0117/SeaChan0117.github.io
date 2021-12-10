---
title: 钉子问题-动态规划
date: 2021-11-03 09:51:06
categories:
- 算法
- 动态规划
tags:
- 动态规划
cover: https://tse4-mm.cn.bing.net/th/id/OIP-C.4F2osfntSKIt-LsCl_IuAwHaE8?pid=ImgDet&rs=1

---

# 问题描述

小明装修需要 n(1<=n<=200) 颗钉子，但是五金店没有散装钉子卖，只有两种盒装包装的，一种包装 4 颗，一种包装有 9 颗，请问小明最少需要买多少盒钉子才能刚好买够 n 颗，无答案时输出 -1？

# 递归

```js
function boxesFn(n) {
    if (n === 0) return 0
    let res = Infinity
    if (n >= 4) {
        res = Math.min(boxesFn(n - 4) + 1, res)
    }

    if (n >= 9) {
        res = Math.min(boxesFn(n - 9) + 1, res)
    }

    return res || -1
}

console.time('boxes')
console.log(boxesFn(100))
console.timeEnd('boxes')
```

# 动态规划

```js
function boxesFn_dy(n) {
    let numArr = [4, 9]
    const res = new Array(n + 1).fill(-1)
    res[0] = 0
    for (let i = 1; i <= n; i++) {
        for (let j = 0; j < numArr.length; j++) {
            if (i >= numArr[j] && res[i - numArr[j]] !== -1) {
                res[i] = Math.min(res[i - numArr[j]] + 1, Infinity)
            }
        }
    }
    return res[n]
}

console.time('boxesFn_dy')
console.log(boxesFn_dy(100))
console.timeEnd('boxesFn_dy')
```
