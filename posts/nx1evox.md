---
title: 蓝狐NX1刷入EVOXA16GSI
date: 2026-02-05
tags: [手机]
head:
  - - meta
    - name: description
      content: 蓝狐NX1刷入EVOXA16GSI
---

不喜欢官方系统一口气隐藏所有状态栏图标的作法？刷了澎湃OS却发现蓝牙耳机经常断连？可以试试这个。

---

# 前言

之前有在油管看到阿哲的视频中有展示蓝狐NX1刷安卓16的EVOX系统，整体的UI设计和状态栏进度条的设计都很亮眼。之后虽然有尝试过刷入各种版本的EVOX的GSI，但因为当时蓝狐社区官方只提供国内版的底包。就算刷进去了也是不能用谷歌的状态(A16)，更何况还有拨打电话时会出现音频服务出错的情况（A15），最致命的是还有IMS的问题，没办法只能暂时放弃。不过随着时间来到2026年，不仅官方已经推出了国际版的底包，酷安也有大佬发了修复音频服务的模块。经过几天的尝试与努力，终于实现了刷入EVOX的同时还能实现正常接打电话、发送短信并且能够正常使用流量上网的状态了。本篇着重记录整个实现过程。以实现正常使用为目的。

::: tip
以下内容仅针对蓝狐NX1+美版系统底包+EVOX_gapps(A16)+中国联通的组合，其他运营商因条件限制未做测试。
:::
 
## 刷机流程

::: tip
解BL操作可以参考蓝狐社区文章或自行搜索相关B站视频。
:::

### 准备工作

在正式刷机前，需要先准备好以下工具及文件：
- 一台电脑
- 充满电的手机
- 数据线
- [官方底包（美版20251029）](https://www.bluefoxforum.com/single/988)
- [官方提供的刷机工具和驱动](https://www.bluefoxforum.com/single/339)
- [EVOXGSI包（arm64-ab）](https://github.com/Doze-off/EvoX_treble/releases/tag/2025-09-02)
- [Fish_Neko发布于酷安的修复模块（只需要下载01B_For_Patched_LineageOS_VolteAudio_Fixup_NX1）](https://www.coolapk.com/feed/68424693?s=MGQwNjQ0MmQyNjExNmFjZzY5N2YzMjVkega1572)
- [sukiSU Ultra3.2](https://sukisu.org/zh/)
- [IMSAPK 提取码：2v9T](https://pan.quark.cn/s/252d94b911f5)


### 底包刷入及init_boot修补
首先需要为电脑安装官方提供的驱动程序并重启电脑。
重启完成后，启动刷机工具中的flash_tool.exe，并在Scatter-loading File中选择官方底包中的MT6768_Android_scatter.txt，最后点击Download。

![Caption](/guanfang.jpg)

::: tip
接下来的操作会导致手机内数据全部清空，继续操作前应确保重要数据已得到备份。
:::

当Download按钮变灰后，在确保手机完全关机的状态下，插入USB线并连接电脑，正常情况下，软件界面将会显示分区烧录进度。

烧录完成后，软件将会弹出完成提示，此时可以直接拔掉USB线。

正常开机并进入系统后，使用USB线连接电脑，并将sukiSU安装包与官方底包的init_boot.img一并传输到手机中。

接着，安装sukiSU 并选择修补init_boot.img。

修补完成后，可以在sukiSU提示的位置中找到修补后的init_boot.img，将其传输到电脑后，关闭手机。


### GSI刷入以及模块安装

在手机保持关机的状态下，按住音量增加键+电源键几秒后，会进入模式选择界面。通过点击音量增加键切换到fastboot模式后，按下音量减小键进入该模式，并将手机通过USB线连接电脑。

接着需要在电脑中启动命令行工具，通过fastboot devices确认手机已正常识别后，输入以下命令：

```bash
fastboot flash init_boot_a 修补后的init_boot.img路径
fastboot reboot fastboot
fastboot flash system_a EVOXGSI的所在路径
fastboot reboot recovery
```  

接着在手机的recovery界面中，通过音量键切换选项到wipe data/factory reset,点击电源键进行选择。在新的提示界面中，选择yes并进行双清。

双清完成后，切换选项到power off并点击电源键执行关机。


::: tip
以下操作需要确保手机卡处于插入状态。
:::

启动手机后，将进入一段时间的初始化状态。初始化完成后，正常执行欢迎流程直到进入桌面。

进入桌面后，安装sukiSU管理器 3.2.0并启动。正常情况下，程序界面应该显示运行在LKM模式，如果没问题，则可以安装酷安大佬发布的修复模块。
模块安装完成后，回到桌面，启动treble程序并按照以下顺序操作：

![Caption](/imssettings.jpg)

```ini
找到AUDIO SETTINGS并点击进入后，启用DISABLE VOICE CALL IN ROUTE并回到主选单。
找到IMS设置并点击进入后，点击一次create ims apn，启用request ims network与force the presence of 4g calling setting。
点击INSTALL IMS APK FOR MEDIATEK R+ VENDOR（这步需要连接wifi操作 下载也会比较慢 也可以选择直接安装文末提供文件中的IMSAPK）
安装完成后，长按手机电源键并选择重启系统
```  

### 收尾测试

重新启动后，对接打电话、收发短信及流量上网测试正常即可视为完成。

![Caption](/zm1.jpg)

### 遇到问题？

已知的：

```ini
不安装IMSAPK会导致IMS注册失败 无法接听电话 短信收发等不受影响。
VANILLA版本无法正常接收通知 无法发短信。
不安装模组会导致音频服务崩溃 无法正常通话。
```  

::: tip
如果需要刷入其他版本，切记登出谷歌账号后再进行操作。
:::

如果刷机过程中遇到任何问题，可直接跳转到使用官方刷入工具的步骤重新烧录。