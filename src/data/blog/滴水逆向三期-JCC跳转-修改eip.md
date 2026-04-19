---
author: INT3
pubDatetime: 2024-09-23T12:58:53+08:00
modDatetime: 2026-04-19T19:04:53.851+08:00
title: 滴水逆向三期-JCC跳转-修改eip
slug: 滴水逆向三期-JCC跳转-修改eip
featured: false
draft: false
tags:
  - 逆向笔记
description: 本文描述了jcc跳转修改eip，以及其他影响eip的指令(如ret 和 call)
---

Jmp修改Eip的指令
![image.png](https://int3imgbed.tos-s3-cn-shanghai.volces.com/i/2026/03/67782ad9d3f4aa77bc213540e23d0513.png)
如果跳转的地址到开始跳转的地址 字节小于128个字节 则要加一个short
jmp（无条件跳转）唯一的作用就是修改eip 唯一影响的就是eip的值
jmp 寄存器/立即数
Call指令
![image.png](https://int3imgbed.tos-s3-cn-shanghai.volces.com/i/2026/03/671cd353f2f555e9238093626108f8d6.png)
call指令执行之后压入(堆栈)Call的下一条地址
ret指令
相当于POP eip
作业 使用jmp call return 跳出去再调回来

![image.png](https://int3imgbed.tos-s3-cn-shanghai.volces.com/i/2026/04/a62c05ac6fd536c31985849a9d0129e1.png)
