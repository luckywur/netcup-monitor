<div align="center">

Netcup Monitor Pro 📊

专为 Netcup RS/VPS 用户打造的智能化流量监控与自动化流控面板

核心功能 • 部署指南 • 配置说明 • 免责声明

<p align="left">
<b>Netcup Monitor Pro</b> 解决了 Netcup 用户最大的痛点：<b>限速（Throttling）后的自动化处理</b>。




通过对接 Netcup 官方 SOAP API，它能精准识别服务器状态，并根据状态自动指挥 qBittorrent 和 Vertex 进行流量规避和恢复，实现真正的“无人值守”。
</p>

</div>

✨ 核心功能

功能模块

详细说明

🛡️ 智能限速策略

HR 保护：限速时，做种中的 HR 种子限速上传（保种），下载中的 HR 种子暂停。



空间释放：非保留分类种子自动删除，保留分类种子自动暂停。

⚡ 自动恢复模式

当 API 检测到限速解除后，自动恢复所有暂停任务，并解除速度限制，无需人工干预。

🔗 Vertex 联动

智能选机：自动将 Vertex 下载服务器列表更新为当前未限速的机器。



容器自愈：规则更新后支持自动重启 Vertex 容器。

📊 监控面板

流量趋势：七日流量消耗可视化图表。



健康报告：记录每日限速时长、日均限速分析。

🔔 多渠道通知

支持 Telegram Bot、企业微信 (Webhook) 和 企业微信 (应用) 推送每日状态简报。

🚀 部署指南 (Docker Compose)

1. 准备工作

确保服务器已安装 Docker 和 Docker Compose。

2. 创建目录与配置

mkdir -p /root/netcup-monitor/data
cd /root/netcup-monitor
nano docker-compose.yml


3. Docker Compose 配置

复制以下内容。注意 ports 部分可自定义访问端口。

version: '3.8'

services:
  netcup-monitor:
    image: ghcr.io/agonie0v0/netcup-monitor:latest
    container_name: netcup-monitor
    restart: unless-stopped
    
    # --- 端口配置 ---
    # 格式: "宿主机端口:容器端口"
    # 例如想用 8080 访问，则改为 "8080:5000"
    ports:
      - "5000:5000"
    
    volumes:
      - ./data:/app/data
      - /etc/localtime:/etc/localtime:ro
      # (可选) Vertex 自动重启功能需要
      - /var/run/docker.sock:/var/run/docker.sock
    
    environment:
      - TZ=Asia/Shanghai


4. 启动服务

docker-compose up -d


访问地址：http://你的IP:5000

⚙️ 初始化配置

首次访问点击右上角 “登录”（默认无密码），进入设置：

基础连接：填写 qBittorrent 和 Vertex 信息。

Netcup SCP 账号：必填。用于 SOAP API 检测限速状态（账号密码仅本地存储）。

策略配置：

保留分类：限速时只暂停不删除的分类（如 Keep）。

HR 保护分类：限速时限制上传速度的分类（如 HDSky）。

⚠️ 免责声明

[!WARNING]
本项目涉及对 qBittorrent 种子的 删除 和 暂停 操作。

请务必正确配置 “保留分类” 及 “不托管” 选项，防止误删重要数据。

作者不对因配置错误导致的数据丢失负责。

<div align="center">
Project maintained by <a href="https://www.google.com/search?q=https://github.com/agonie0v0">agonie0v0</a>
</div>
