---
title: 浅析诺瓦泰OEM718D
date: 2023-12-14
tags: [硬件, GPS, RTK]
head:
  - - meta
    - name: LVTTL和232，到底长什么样？
      content: LVTTL和232，到底长什么样？
  - - meta
    - name: LVTTL和232，到底长什么样？
      content: LLVTTL和232，到底长什么样？
---

这块卡虽然价格昂贵，但性能和稳定性的确是实打实的。

---

# 前言

由于之前的GPS模块存在丢星问题且不具备天线定向功能，遂尝试订购这块板卡作为替换模块。  
![Caption](/718D.jpg)
## 板卡引脚信息
下面的列表是板卡的引脚定义，另一侧是两个焊盘上的天线底座（比较脆弱，容易脱焊）。  
![Caption](/718D3.jpg)  
![Caption](/718D4.jpg)

## 板卡配置信息
板卡默认波特率9600、8n1，使用novatel connect即可进入配置模式。  
在软件首页可以根据显示数据判断主天线，没记错的话应该是插针视角右侧天线为主。  
在第二页配置你需要的波特率、定位信息更新频率（最高只能设置到5，原因未知）以及RTK类型以及差分数据串口（此处类型为RTCM3 上下都是）。
![Caption](/718D2.jpg)