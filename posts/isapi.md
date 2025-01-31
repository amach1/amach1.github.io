---
title: 树莓派搭配海康威视枪机通过ISAPI获取快照
date: 2024-12-18
tags: [硬件, 树莓派]
head:
  - - meta
    - name: 树莓派
      content: 树莓派
  - - meta
    - name: 树莓派
      content: 树莓派
---

目前只能保存图片。 

---

### 配置网络参数

为了方便后续连接wifi后不出问题，这里需要设置IP为一个确定不会和wifi冲突的网段（以10.0.0.1作为示例），确认其他参数无误后，点击保存。
![Caption](/ipcam1.jpg) 

## 树莓派配置

设置树莓派RJ45口为自己需要的地址。 
![Caption](/ipcam3.jpg) 


## 测试

关闭配置页面（或者关闭所有浏览器，防止当前会话的cookie影响后续测试）。
浏览器输入http://<ID>:<PW>@<IP>/ISAPI/Streaming/channels/101/picture?snapShotImageType=JPEG
如果一切顺利，你应该能看到浏览器当前捕获的图像。
如果浏览器未加载任何元素，检查你是否还有未关闭的配置页面或任何登陆了摄像头管理账户的页面。
如果浏览器弹出登录窗口，不要继续刷新。确认你的账号密码是否正确（除非二者一样，否则不要在凭据输入窗口输入摄像头管理账户，这一步需要验证的是你在onvif配置页面中创建的ID和PW）

## 使用
使用wget方法下载图片
wget -O camera.jpeg 'http://<ID>:<PW>@<IP>/ISAPI/Streaming/channels/101/picture?snapShotImageType=JPEG'