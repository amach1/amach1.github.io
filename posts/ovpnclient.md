---
title: CentOS服务器部署OpenVPN服务端并启用网对网（客户端篇）
date: 2025-02-05
tags: [服务器, VPN]
head:
  - - meta
    - name: description
      content: CentOS服务器部署OpenVPN服务端并启用网对网（客户端篇）
---

比N2N兼容性高、比WireGuard覆盖面广。

---

# 前言
记录下OpenVPN客户端的部署流程（Debian）
目标是实现客户端A与客户端B的LAN设备可互访，且客户端A、B下的LAN设备可访问其他客户端，但禁止其他客户端直接访问客户端A、B下的LAN用户。

::: tip
本文仅包含OpenVPN客户端配置。
服务端配置及相关内容会在另一篇中说明。
:::
 
## 服务端侧准备文件
在正式准备客户端前，需要先在服务端准备好客户端正常通信所需的证书等文件。

### 生成客户端证书
```bash
cd ~/openvpn-ca
./easyrsa build-client-full 客户端名称 nopass
cp ~/openvpn-ca/pki/ca.crt ~/client-config/
cp ~/openvpn-ca/pki/issued/客户端名称.crt ~/client-config/
cp ~/openvpn-ca/pki/private/客户端名称.key ~/client-config/
``` 

### 客户端路由文件

文件应存储在服务端配置文件中client-config-dir的路径 
以 /etc/openvpn/ccd 为例，以下是一个配置文件样例，建议根据自身需求修改

::: tip
需要确保客户端文件名与客户端证书名完全一致如： /etc/openvpn/ccd/client1
仅有网对网客户端需要进行此操作，或者需要对某一客户端进行静态IP设置。
:::

```ini
# 客户端虚拟IP设置
ifconfig-push 10.123.0.2 255.255.255.0

# 客户端本地LAN网段
iroute 192.168.0.0 255.255.255.0

# 除本机外的其他客户端分支推送（如果有网对网需求）(并非全局声明)
push "route 192.168.1.0 255.255.255.0" # 客户端路由器2
push "route 192.168.50.0 255.255.255.0" # 客户端路由器3
push "route 192.168.123.0 255.255.255.0" # 客户端路由器4
```  


## 客户端前期配置（有网对网需求）

以华硕AC56路由器为例

### 准备ovpn配置文件 “客户端名称.ovpn”

```ini
# profile-version 1


client
dev tun
proto udp

remote 服务端IP 服务端端口
resolv-retry infinite
nobind
persist-key
persist-tun

pull  # 明确是网对网

cipher AES-256-GCM  # 加密
auth SHA256

verb 3

<ca>
 ## 把ca.crt的内容粘贴到这里
</ca>

<cert>
 ## 把客户端名称.crt的内容粘贴到这里
</cert>

<key>
 ## 把客户端名称.key的内容粘贴到这里
</key>
```  

### 路由器配置

准备好客户端名称.ovpn后，进入路由器后台管理页面，找到VPN菜单，选择OpenVPN选项卡，新建配置名称并将后缀为ovpn的文件上传保存后，启用配置即可。


### 测试

可优先尝试使用路由器ping服务端及其他客户端，确认可行后再逐步ping其他客户端LAN下用户。

## 客户端前期配置（没有网对网需求）

以Debian12为例（OpenVPN最好以root权限运行）

### 安装

```bash
sudo apt update
sudo apt install openvpn -y
```  

### 传输客户端所需文件

将服务端中生成的 ca.crt、客户端名称.crt与客户端名称.key 保存到客户端的 /etc/openvpn/client/ 目录下。

### 创建客户端配置文件
```bash
sudo vim /etc/openvpn/client/client.conf
```  

配置文件内容

以下是一个配置文件样例，建议根据自身需求修改

```ini
client
dev tun
proto udp
remote 服务端IP 服务端端口
resolv-retry infinite
nobind
persist-key
persist-tun
ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/客户端名称.crt
key /etc/openvpn/client/客户端名称.key
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
topology subnet
verb 3
```

### 启动OpenVPN客户端
```bash
sudo systemctl start openvpn@client
sudo systemctl enable openvpn@client
```

### 验证VPN连接状态
```bash
ip addr show tun0  # 应显示分配的虚拟IP如10.123.0.5
ping 10.123.0.1      # 测试连通性
```

### 确认SSH配置监听（如果有使用OpenVPN访问本机SSH的需求）
```bash
sudo vim /etc/ssh/sshd_config
```
添加或者修改为
```ini
ListenAddress 0.0.0.0  
```
保存后重启
```bash
sudo systemctl restart ssh
```