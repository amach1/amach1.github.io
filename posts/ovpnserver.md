---
title: centos服务器部署OPENVPN服务端并启用网对网（服务端篇）
date: 2025-02-5
tags: [服务器, vpn]
head:
  - - meta
    - name: description
      content: centos服务器部署OPENVPN服务端并启用网对网（服务端篇）
---

比N2N兼容性高、比WireGuard覆盖面广。

---

# 前言
记录下OpenVPN服务的部署流程（centos）。
目标是实现客户端A与客户端B的LAN设备可互访，且客户端A、B下的LAN设备可访问其他客户端，但禁止其他客户端直接访问客户端A、B下的LAN用户。

::: tip
本文仅包含OpenVPN服务端基础部署与客户端互通配置。
客户端配置及相关内容会在另一篇中说明。
:::
 
## 服务端前期配置

### 启用EPEL仓库
```bash
sudo yum install epel-release -y
```  

### 安装OpenVPN和Easy-RSA
```bash
sudo yum install openvpn easy-rsa -y
```  

### 初始化并生成证书与参数、复制文件
```bash
mkdir -p ~/openvpn-ca
cp -r /usr/share/easy-rsa/3/* ~/openvpn-ca
cd ~/openvpn-ca
./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa build-server-full server nopass
./easyrsa gen-dh
```  


### 创建证书目录及服务端配置文件
```bash
sudo mkdir /etc/openvpn/server
sudo cp ~/openvpn-ca/pki/ca.crt /etc/openvpn/server/
sudo cp ~/openvpn-ca/pki/issued/server.crt /etc/openvpn/server/
sudo cp ~/openvpn-ca/pki/private/server.key /etc/openvpn/server/
sudo cp ~/openvpn-ca/pki/dh.pem /etc/openvpn/server/
sudo vim /etc/openvpn/server/server.conf
```  

配置文件内容

以下是一个配置文件样例，建议根据自身需求修改

```ini
port 1234 # 建议自主选择端口
proto udp 
dev tun 
ca ca.crt 
cert server.crt 
key server.key 
dh dh.pem 

server 10.123.0.0 255.255.255.0  # VPN 地址池 建议自主选择 

client-to-client # 允许隧道转发

client-config-dir /etc/openvpn/ccd  # 客户端路由声明目录 

keepalive 10 120 

persist-key 
persist-tun 
topology subnet 

status openvpn-status.log  # 日志记录 
verb 3 
log-append /var/log/openvpn.log
```

### 网络配置（记得后台开启对应端口）
```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --permanent --add-port=1234/udp # 注意端口
sudo firewall-cmd --permanent --add-masquerade # 视实际情况决定
sudo firewall-cmd --reload
sudo systemctl start openvpn-server@server
sudo systemctl enable openvpn-server@server
mkdir ~/client-config
```  

::: tip
可以使用这个命令确认下是否正确 sudo firewall-cmd --list-ports
:::
确认没问题后，就可以尝试配置客户端了。