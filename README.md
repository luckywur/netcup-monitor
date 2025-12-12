# Netcup Monitor Pro 📊

![Docker](https://img.shields.io/badge/Docker-Enabled-blue?logo=docker)
![Python](https://img.shields.io/badge/Python-3.9+-yellow?logo=python)
![Status](https://img.shields.io/badge/Status-Active-success)

一个专为 Netcup VPS 用户打造的高级监控与自动化管理面板。集成了流量统计、限速检测、自动刷流策略控制以及 Vertex 联动功能。

## ✨ 核心功能

* **📈 实时/历史流量统计**：
    * 直观展示今日、本月及日均流量消耗。
    * 支持多台服务器集中管理。
    * 七日流量趋势图表可视化。
* **🛡️ 智能限速检测 (SOAP API)**：
    * 直接对接 Netcup SCP 接口，精准判断服务器是否被限速。
    * 记录限速历史时长，生成健康度报告。
* **🤖 自动化 qBittorrent 管理**：
    * **限速模式**：自动暂停刷流种子，仅保留指定分类（如 PTER, FRD），并自动删除非保留分类的种子以释放空间。
    * **恢复模式**：检测到限速解除后，自动恢复所有种子下载/上传。
* **🔗 Vertex 深度联动**：
    * 自动从 Vertex 获取最新的 RSS 任务。
    * 根据当前“幸存”的高速服务器列表，动态更新 Vertex 的任务配置（只让没限速的机器刷流）。
* **📱 消息推送**：
    * 支持 Telegram Bot 推送每日状态简报。
* **💻 现代化 Web 面板**：
    * 基于 Bootstrap 5 设计，响应式布局（适配手机端）。
    * 支持 Web 端直接修改配置、管理账号。

## 🚀 快速部署 (Docker Compose)

### 1. 准备环境
确保你的服务器已安装 Docker 和 Docker Compose。

### 2. 创建配置文件

mkdir -p /root/netcup-monitor/data

cd /root/netcup-monitor

并在其中创建 `docker-compose.yml` 文件：

```yaml
version: '3.8'

services:
  netcup-monitor:
    image: ghcr.io/agonie0v0/netcup-monitor:latest
    container_name: netcup-monitor
    restart: unless-stopped
    network_mode: "host"
    
    volumes:
      - ./data:/app/data
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock
    
    environment:
      - TZ=Asia/Shanghai
