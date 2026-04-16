---
author: INT3
pubDatetime: 2022-09-26T12:13:24Z
modDatetime: 2024-01-04T09:09:06Z
title: 超级HOOK获取寄存器数据(易语言)
slug: 超级HOOK获取寄存器数据(易语言)
featured: false
draft: false
tags:
  - 易语言
description: 超级HOOK获取寄存器数据(易语言)
---

那段时间跟着乐易的重命名在学习易语言HOOK示例，买了他几百块的课程，也是很受益匪浅的，他讲课不是浮于表面，会告诉你为什么这样做。
<img width="1905" height="942" alt="image" src="https://github.com/user-attachments/assets/7ec5b4ec-ff4d-4cc5-8de0-92ce16259a2a" />

还买了专门写辅助的课程但是好像下架了,偶然看到了超级HOOK的写法，感觉有点好奇和震惊，原来画方框也是可以不用找基地址的，直接一句超级HOOK.开始HOOK()就可以得到寄存器的值，在我学习hook之后，萌生了自己写一个类似的工具，在网上搜索之后发现已经有大佬写出来了
<img width="1165" height="799" alt="image" src="https://github.com/user-attachments/assets/e6ba9dd5-3b9e-4202-ab80-7006c8e26653" />

但是本着第一次学习和小试牛刀的心态，我依旧决定自己写一个。
<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/040c35bd-a08a-48b8-b750-0819ebb472ef" />

其实是大差不差的，但是我是学习完阿荒的源码，在OD和CE一边试，一边研究加备注，所以你可以看到我们在变量的命名和注释都是比较详细的，包括我现在看这份代码也不是很清晰(很久没碰了)。
大致原理概述一下：
找到你需要HOOK的地址的地方，必须大于等于5个字节，因为jmp+一个32位的地址已经是5个字节了，如果原来不足五个字节，我们写入6个字节，之后的代码就会受到影响，具体可以看一下OD随便改一条指令看一下。

`回调位置 ＝ HOOK地址 ＋ HOOK字节数 原始函数+它本身的字节数就是他的回调地址`

这句话的意思就是我们HOOK的地址加上他自身原来的字节长度就是我们要跳回来的地方

`原始数据 ＝ 内存_读字节集 (进程ID2, HOOK地址, HOOK字节数)`

原始数据是什么意思呢，就是我们保存的原始数据，HOOK完跳转回来的时候需要重新写入回调地址。
`内存_写字节集 (进程ID2, HOOK地址, { 233 } ＋ 到字节集 (到整数 (空白内存地址 － HOOK地址 － 5)) ＋ nop补码 (HOOK字节数))`
开始jmp跳转 空白地址 就是申请的一片为0的没有程序访问的空间 这里我们采用的是相对跳转，即
`0xE9 + 4字节偏移量 （32 位模式），偏移量 = 目标地址 - (当前 jmp 地址 + 5)+ nop补码 (HOOK字节数))`
这里的5是排除当前这条指令的长度 当然也有跳转绝对地址的办法，对于平坦内存模型（32 位保护模式，如 Windows/Linux 用户态），常用 `0xFF 0x25` 格式直接跳转到绝对地址。绝对跳转无需计算偏移量，适合目标地址固定的场景，但指令更长。相对跳转更节省空间（5 字节），是 HOOK 中更常用的方式（尤其是原指令长度为 5 字节时，无需额外处理长度）。nop补码是如果原指令是7个字节，我们这里是5字节，为了7>5，所以需要补nop，不是0哦，补齐7个字节。
<img width="364" height="147" alt="image" src="https://github.com/user-attachments/assets/a36a7430-9924-479b-ae05-e030ff47897b" />
这里的作用是在空白地址存储等会要记录的寄存器的数据
<img width="915" height="189" alt="image" src="https://github.com/user-attachments/assets/196a97be-62a4-49fc-8f26-b18c7ce4fd13" />
间隔每个指令都是一样的 同之前一样 把各个寄存器的值写入不同的内存单元 然后写入原始数据 最后在jmp回去的偏移地址 = 回调地址[HOOK地址的下一条地址] - 我们在空白地址写入那一片的地址长度
有人可能会问了，你之前是申请的空白地址-hook地址 在空白区域最后一条指令是回调地址 - 空白内存地址，这到底哪个大哪个小？
答：判断方向的核心规则
偏移量为负数：表示往前跳（跳向低地址方向，目标地址在当前指令之前）。
偏移量为正数：表示往后跳（跳向高地址方向，目标地址在当前指令之后）。
也就是说jmp是有符号数，前面有标记是正负。
整个流程基本就完事了，简单来说，就是修改原始指令，跳转到一块空白内存，然后写入mov [address],eax 等类似的指令，然后继续执行被HOOK的指令，最后跳转到hook地址的下一条指令继续执行。
顺便贴一份评论区大佬的无模块源码，这备注，确实清晰
<img width="1487" height="687" alt="image" src="https://github.com/user-attachments/assets/4a948d99-e83b-4095-91d0-06c6adfe798e" />

我也是很久没看这份代码了，今天想着分享，顺便浏览了一下，加深了一下学习，共勉！
最后上一下实际用途的截图[获取了cs起源人物的血量]
<img width="1360" height="728" alt="image" src="https://github.com/user-attachments/assets/9abfed36-a305-437e-b690-951833592385" />
<img width="1360" height="767" alt="image" src="https://github.com/user-attachments/assets/d08cf1da-daf0-4f85-acd1-c978c720402b" />
<img width="1360" height="768" alt="image" src="https://github.com/user-attachments/assets/897a395b-b8f2-430a-a5dc-0e30949ec3fb" />

下载地址1（Onedrive）:<p>[超级HOOK获取寄存器数据(易语言)](https://int3666-my.sharepoint.com/:u:/g/personal/int3_int3666_onmicrosoft_com/IQBObzMHzzY7QIThMGl2Vxk3AfbrU-IrQqjV_4xnRVEyE-Y?e=Ka3k4x)</p>

下载地址2（蓝奏云）:<p>[超级HOOK获取寄存器数据(易语言)](https://shenxiu.lanzout.com/irTiD1n2c8jg%20%E5%AF%86%E7%A0%81:djd5)</p>

