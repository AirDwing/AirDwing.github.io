---
title: "Willin: Node.js CPU调度优化"
date: 2017-06-14 12:45:36
categories: Dev
tags: [node.js, azure, iothub, eventhubs]
---

{% cq %}
Master / Cluster 模式
{% endcq %}

## 单一服务器多核心分配

假设处理的任务列表如下:

```js
const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0];
```

以10为例,假设服务器为4CPU,那么每个CPU处理的任务分别为:

- CPU1: [1, 2, 3]
- CPU2: [4, 5, 6]
- CPU3: [7, 8]
- CPU4: [9, 0]

<!--more-->

```js
const numCPUs = require('os').cpus().length; // 假设该值为 4

// 处理的任务列表
const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0];

// 调度处理代码写在这儿
// 每个 CPU 分配 N 个任务
const n = Math.floor(arr.length / numCPUs);
// 未分配的余数
const remainder = arr.length % numCPUs;

for (let i = 1; i <= numCPUs; i += 1) {
  console.log(arr.splice(0, n + (i > remainder ? 0 : 1)));
}
```

## 多服务器多核心分配调度

假设处理的任务列表如下:

```js
const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32];
```

有多台负载均衡器,不确定服务器数量和核心数量.


假设目前有3台服务器,分别为 `4` 核心, `6` 核心, `8` 核心.

按照核心数进行优先调度,那么每个CPU处理的任务分别为:

- 服务器1 (`4` 核心)
  - CPU1: [ 29 ]
  - CPU2: [ 30 ]
  - CPU3: [ 31 ]
  - CPU4: [ 32 ]
- 服务器2 (`6` 核心)
  - CPU1: [ 17, 18 ]
  - CPU2: [ 19, 20 ]
  - CPU3: [ 21, 22 ]
  - CPU4: [ 23, 24 ]
  - CPU5: [ 25, 26 ]
  - CPU6: [ 27, 28 ]
- 服务器3 (`8` 核心)
  - CPU1: [ 1, 2 ]
  - CPU2: [ 3, 4 ]
  - CPU3: [ 5, 6 ]
  - CPU4: [ 7, 8 ]
  - CPU5: [ 9, 10 ]
  - CPU6: [ 11, 12 ]
  - CPU7: [ 13, 14 ]
  - CPU8: [ 15, 16 ]

```js
const numCPUs = require('os').cpus().length; // 假设该值为 4

// 处理的任务列表
const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32];

// 调度处理代码写在这儿
```

未完待续
