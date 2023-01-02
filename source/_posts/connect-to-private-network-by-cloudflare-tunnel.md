---
title: 基于 Cloudflare Tunnel 实现内网穿透
date: 2023-01-02 21:39:43
header-img: "/images/header_bg/network-server.jpg"
tags:
  - network
  - 内网穿透
---
# 基于 Cloudflare Tunnel 实现内网穿透
`Cloudflare Tunnel` 可以像 `frp` 一样实现内网穿透，特别适用于想公网访问宽带网络或者无固定IP的私有网络。`frp` 需要一台具有公网IP的服务器作为Server端来转发流量，`Cloudflare Tunnel`则无需你自己购买这样的服务器，利用免费的 `Cloudflare Edge Server` 即可实现内网穿透，对于个人开发者或者轻度使用非常友好。

[How Cloudflare Tunnel Works](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/#how-it-works)

### 服务端配置（被公开的服务器）

1. Setup `Cloudflared`
```sh
# Mac 版本
brew install cloudflare/cloudflare/cloudflared

# 其他版本
# https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/#set-up-a-tunnel-locally-cli-setup
```
2. Login
```sh
# 登录cloudflare，成功后会生成 ~/.cloudflared/cert.pem
# cloudflare需要配置自己的域名并创建一个应用（主要用于后续做DNS映射）
cloudflared tunnel login
```
3. Create a tunnel
```sh
# 创建tunnel，请记住tunnel的uuid，记不住可以使用 cloudflared tunnel list
cloudflared tunnel create vpc
```
4. Create configuration file
```yaml
# ~/.cloudflared/config.yml
# 以下配置文件做了两个服务的映射，一个ssh，一个http站点
ingress:
  - hostname: ssh.bomee.xyz
    service: ssh://localhost:22
  - hostname: html.bomee.xyz
    service: http://localhost:8080
  - service: http_status:404
tunnel: 1c025733-a2ec-4ec5-8d3a-9c9d6775e49b # 上面创建的tunnel id
credentials-file: ~/.cloudflared/1c025733-a2ec-4ec5-8d3a-9c9d6775e49b.json # 登录后的认证文件
```
5. Create DNS route
```sh
cloudflared tunnel route dns 1c025733-a2ec-4ec5-8d3a-9c9d6775e49b ssh 
cloudflared tunnel route dns 1c025733-a2ec-4ec5-8d3a-9c9d6775e49b html 
```
6. Run
```sh
cloudflared tunnel run
```

### 客户端配置（访问的主机仅ssh需要）
```sh
# 编辑ssh配置 vim ~/.ssh/config
# 客户端也需要安装cloudflard
Host ssh.bomee.xyz
  ProxyCommand /usr/local/bin/cloudflared access ssh --hostname %h

# ssh
ssh bomee@ssh.bomee.xyz
```