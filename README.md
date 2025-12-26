<div align="center">

📊 Netcup Monitor Pro

专为 Netcup RS/VPS 用户打造的智能化流量监控与自动化流控面板

功能特性 • 工作原理 • 部署指南 • 配置详解 • 界面预览

</div>

📖 项目简介

Netcup Monitor Pro 是一个针对 Netcup 服务器流量限制（Throttling）痛点设计的自动化运维工具。

不同于普通的本地流量监控脚本，本项目通过对接 Netcup 官方 SOAP API，精准识别服务器当前的“限速状态”。当检测到服务器被限速时，它能接管您的下载器（qBittorrent）和生态工具（Vertex），执行精细化的流控策略，并在限速解除后自动恢复生产，实现真正的全自动无人值守。

✨ 核心功能

🛡️ 智能流控策略

HR 智能保护：

下载中的 HR 种子：限速期间自动暂停，防止无效流量消耗。

做种中的 HR 种子：限速期间自动限制上传速度（如 10KB/s），确保种子处于活动状态，防止因无流量被站点判定为 H&R。

分类管理机制：

保留分类 (Keep)：自定义保留分类，限速时仅暂停，绝不删除文件。

自动清理：非保留、非 HR 分类的普通种子，在限速时自动删除以释放宝贵的磁盘空间。

自动恢复：检测到限速解除（恢复高速）后，自动恢复所有暂停任务并解除速度限制。

🔗 Vertex 生态深度集成

智能选机：与 Vertex 联动，根据服务器限速状态动态更新 RSS 下载规则。只将任务分发给未限速的机器。

容器自愈：规则更新后支持通过 Docker Socket 自动重启 Vertex 容器，确保配置即时生效。

📊 可视化仪表盘

实时监控：在 Web 界面查看所有实例的实时上传/下载速度、今日流量、本月流量。

趋势分析：内置 7 日流量消耗折线图与服务器健康度（限速时长）分析。

Web 热配置：无需重启容器，所有参数（账号、策略、通知）均可在网页端实时修改。

🔔 多渠道通知

支持 Telegram Bot、企业微信（Webhook） 及 企业微信应用（App） 推送，实时掌握服务器状态变更。

🧠 自动化策略逻辑

系统默认每 5 分钟 检测一次状态，根据 Netcup API 返回的 trafficThrottled 状态执行以下逻辑：

种子类型 / 分类

🟢 高速状态 (Normal)

🔴 限速状态 (Throttled)

HR (做种中)

🚀 全速上传

🛡️ 限速上传 (默认 10KB/s, 防掉种)

HR (下载中)

🚀 全速下载

⏸️ 暂停下载

保留分类 (Keep)

▶️ 恢复运行

⏸️ 暂停任务 (保留文件)

普通种子

▶️ 恢复运行

🗑️ 直接删除 (释放空间)

全局限速

🔓 无限制

🔒 全局限速生效

🚀 快速部署

推荐使用 Docker Compose 进行一键部署。

1. 准备工作

确保服务器已安装 Docker 和 Docker Compose。

2. 创建配置文件

创建项目目录并编写 docker-compose.yml：

mkdir -p /root/netcup-monitor/data
cd /root/netcup-monitor
nano docker-compose.yml


填入以下内容：

version: '3.8'

services:
  netcup-monitor:
    image: ghcr.io/agonie0v0/netcup-monitor:latest
    container_name: netcup-monitor
    restart: unless-stopped
    # 端口映射 (宿主机端口:容器端口)
    ports:
      - "5000:5000"
    volumes:
      # 数据与配置文件持久化
      - ./data:/app/data
      # 宿主机时间同步
      - /etc/localtime:/etc/localtime:ro
      # (可选) 若需自动重启 Vertex 容器，必须挂载此 Socket
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TZ=Asia/Shanghai
      # - SECRET_KEY=your_secret_key  # 可选：用于 Session 加密，不填会自动生成


3. 启动服务

docker-compose up -d


服务启动后，访问面板：http://你的IP:5000

⚙️ 配置详解

首次访问点击右上角 “登录”（默认无密码），进入设置页面。

1. 基础连接

qBittorrent 设置：填写 QB 的地址、账号、密码。

Vertex 设置：

API 地址：Vertex 的访问地址 (如 http://192.168.1.2:3000)。

容器名：如果 Vertex 和本服务在同一台机器，填写 Vertex 的容器名（如 vertex）可实现自动重启。

监控 RSS ID：在 Vertex 规则列表中找到需要动态管理的规则 ID（如 4c145005），多个 ID 用逗号分隔。

2. Netcup SCP 账号 (核心)

在“SCP 账号”标签页添加账号，这是判断限速状态的关键。

Customer ID：Netcup 客户号（纯数字）。

Password：登录 Netcup 面板的密码。

🔒 安全提示：密码仅保存在本地 data/config.json 中，通过 Docker 隔离，不会上传至任何第三方。

3. 策略管理

保留分类 (Keep Categories)：填写分类名，如 Keep, FRD, PTER。这些分类下的种子在限速时只会暂停，不会被删除。

HR 保护分类：填写如 HR, VIP。

限速上传限制：设置限速期间 HR 种子的最大上传速度（KB/s），推荐 10 KB/s。

4. 实例配置 (Servers)

添加你需要监控的 VPS 实例：

名称：自定义别名 (如 RS-1000-1)。

IP 地址：必须与 Netcup 后台显示的 IP 一致（用于匹配 API 数据）。

Client ID：对应 Vertex 中下载器的名称/ID（用于更新 RSS 规则）。

不托管 (Unmanaged)：开启后，脚本只监控流量和状态，不执行任何删除、暂停操作，也不参与 Vertex 联动。

📸 界面预览

仪表盘概览

流量统计





直观展示限速时长与服务器健康度

详细记录单日、单月流量消耗

⚠️ 免责声明

数据安全：本项目涉及对 qBittorrent 种子的 删除 操作。请务必在正式使用前正确配置 “保留分类” 及 “不托管” 选项，建议先在测试机上运行。

账号安全：Netcup 账号密码存储在您的服务器本地，请确保服务器安全。

责任界定：作者不对因配置错误导致的数据丢失、账号问题或 Netcup 官方政策变动导致的损失负责。

<div align="center">
Project maintained by <a href="https://www.google.com/search?q=https://github.com/agonie0v0">agonie0v0</a>
</div>
