---
title: 使用北醒TF系列雷达替代佳明LIDAR-Lite v3

date: 2023-09-05
tags: [硬件, 雷达 , Arduino]
head:
  - - meta
    - name: 使用北醒TF系列雷达替代佳明LIDAR-Lite v3
      content:  使用北醒TF系列雷达替代佳明LIDAR-Lite v3
  - - meta
    - name: 使用北醒TF系列雷达替代佳明LIDAR-Lite v3
      content: 使用北醒TF系列雷达替代佳明LIDAR-Lite v3
---

由于某不知名飞控仅支持部分国外市售激光雷达，但采购又贵又慢，故产生了本篇以更易于获取的的国产雷达+arduino代替lidar lite v3的记录。

---

# Lidar lite v3工作模式
![Caption](/LITEV3.jpg)
> When space and weight requirements are tight, the LIDAR-Lite v3 soars. It’s the ideal compact, high-performance optical distant measurement sensor solution for drone, robot or unmanned vehicle applications. Using a single chip signal processing solution along with minimal hardware, this highly configurable sensor can be used as the basic building block for applications where small size, light weight, low power consumption and high performance are important factors.  
Featuring all of the core features of the popular LIDAR-Lite v2, this easy-to-use 40 meter laser-based sensor uses about 130 milliamps during an acquisition. It is user-configurable so you can adjust between accuracy, operating range and measurement time.  It’s reliable, powerful ranging and it’s the proximity sensor you need.  
——摘自佳明官网
  
  设备默认以I2C形式输出高度数据，但提供了一个模式控制线来实现pwm输出。

  ![Caption](/LITEV32.jpg)  
    
# 北醒TF系列雷达工作模式
![Caption](/TF02.jpg)    
  

查看相应型号激光雷达的官方文档，这里以TF02pro为例：  


![Caption](/TF022.jpg)   

用串口模块连接激光高度计核对通讯设置以及数据位，并根据数据数量创建结构体并输出。  

# 实现方式 
使用arduino等单片机的串口获取激光高度计数据，并使用PWM引脚输出相应比例脉宽来模拟目标替换设备。

## 具体过程
Arduino nano 与TF-02pro组合的代码（TF-02Pro在2米状态下以该种形式输出脉宽对应的距离为2.072米）

```cpp
  
  typedef struct {
    int distance;
    int strength;
    int temp;
    boolean receiveComplete;
  } TF; 
  TF Lidar = {0, 0, 0, false}; 
  
  void getLidarData(TF* lidar) {
    static char i = 0; // 记录接收数据索引
    char j = 0; // 循环计数
    int checksum = 0; // 校验和
    static int rx[9]; 
    if (Serial.available()) {
      rx[i] = Serial.read(); 
      if (rx[0] != 0x59) { // 检查是否为0x59
        i = 0;
      }
      else if (i == 1 && rx[1] != 0x59) {
        i = 0;
      }
      else if (i == 8) {
        for (j = 0; j < 8; j++) { // 计算数据校验和
          checksum += rx[j];
        }
        if (rx[8] == (checksum % 256)) { // 检查数据校验和
          lidar->distance = rx[2] + rx[3] * 256; // 距离值存储
          lidar->strength = rx[4] + rx[5] * 256; // 强度值存储
          lidar->temp = (rx[6] + rx[7] * 256) / 8 - 256; // 温度值存储
          lidar->receiveComplete = true;
        }
        i = 0;
      }
      else {
        i++; 
      }
    }
  }
  
  void setup() {
    Serial.begin(115200); 
    pinMode(9, OUTPUT); // 9设置为输出
  }
  
  void loop() {
    getLidarData(&Lidar); 
    if (Lidar.receiveComplete) {
      Lidar.receiveComplete = false; 
      digitalWrite(9,HIGH); 
      delayMicroseconds(Lidar.distance*10); // 延迟为距离值乘以10微秒
      digitalWrite(9,LOW); 
      delayMicroseconds(Lidar.distance*10); 
      Serial.print(Lidar.distance); // 打印距离值
      Serial.println(); 
    }
  }
```
烧录代码后，将雷达与arduino连接，并通过串口查看输出距离，当确定距离值与实际情况接近后，使用示波器探针连接输出脚检查脉宽。 

## 与飞控的硬件连接
先看看官方的连接示意图  

![Caption](/2128.jpg)   
高度数据通过AGL接插件蓝色线传输。直接将其与arduino的D9连接。激光高度计的供电部分由arduino提供，arduino的供电来自舵机板的5V供电。
作为MCU以及单片机的一般通讯协议。LVTTL在无数据通讯时往往维持在3.3V左右的的电压范围。  

## 软件设置
  打开VRS的传感器选项卡,将AGL部分如下图设置  
  ![Caption](/AGL.jpg)  

检查飞控字段，此时可以正常显示高度数据。



