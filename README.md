# 🤖 AstrBot + NapCat 极速部署：QQ 智能聊天机器人 (无缝接入 DeepSeek)

欢迎来到本项目！🎉 这是一个基于 **Docker** 容器化部署，结合 **[AstrBot](https://github.com/Soulter/AstrBot)**（机器人大脑，强大的大语言模型框架）与 **[NapCat](https://github.com/NapNeko/NapCatQQ)**（高稳定性 QQ 协议端），为你量身打造的 QQ 智能聊天机器人项目。

本项目目前已完美接入超聪明的 **DeepSeek** 大模型，能够轻松实现多轮对话、逻辑推理等高阶玩法！🚀

---

## ✨ 功能特性

* 🐳 **容器化部署**：基于 Docker Compose，环境隔离，极速一键部署。
* 🧠 **大模型接入**：原生支持 DeepSeek 等主流大模型 API。
* ⚡ **稳定高效**：采用 NapCat 协议端，消息收发极速稳定。
* 🔧 **高度可定制**：AstrBot 强大的插件生态，想怎么玩就怎么玩。

---

## 🛠️ 环境准备

在开始之前，请确保你准备好了以下“装备”：

1. **一个小号**：强烈建议准备一个专门用于机器人的 QQ 小号。
2. **Docker 环境**：推荐使用 [Docker Desktop](https://www.docker.com/products/docker-desktop/) (👈点击前往官网下载)，或者在服务器中安装 Docker Engine。
3. ⚠️ **【踩坑预警 1】：Windows 用户请特别注意！**
   * 如果你在 Windows 下运行 Docker Desktop，**必须**提前安装并启用 **WSL2**（Windows Subsystem for Linux 2），否则 Docker 核心将无法正常启动！(在 PowerShell 中以管理员身份运行 `wsl --install` 即可)。

---

## 🚀 极速部署流程 (保姆级手把手教学)

### 第一步：创建编排文件

在你电脑（或服务器）里新建一个专门存放机器人的文件夹，然后创建一个名为 `docker-compose.yml` 的文件，将以下配置完整复制进去：

```yaml
version: '3.8'

services:
  # --- 机器人核心大脑：AstrBot ---
  astrbot:
    image: soulter/astrbot:latest
    container_name: astrbot
    restart: always
    ports:
      - "6185:6185" # AstrBot WebUI 管理后台端口
      - "6199:6199" # 与 NapCat 通信的反向 WebSocket 端口
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./astrbot_data:/AstrBot/data

  # --- QQ 协议端：NapCat ---
  napcat:
    image: mlikiowa/napcat-docker:latest
    container_name: napcat
    restart: always
    ports:
      - "3000:3000"
      - "3001:3001"
      - "6099:6099" # NapCat WebUI 管理后台端口
    environment:
      # 在 Linux/WSL 环境中获取你的 UID 和 GID 以解决权限问题
      - NAPCAT_UID=1000
      - NAPCAT_GID=1000
    volumes:
      - ./napcat_data:/app/.config/QQ
      - ./napcat_config:/app/napcat/config
```

### 第二步：一键启动容器

在文件所在目录打开终端/命令行（打开方式也很简单，只需要在文件所在的地址处输入 `cmd` 就可以啦），输入：
```bash
docker-compose up -d
```
等待进度条拉满，两个核心服务就在后台乖乖运行啦！✨

### 第三步：给机器人注入灵魂 (AstrBot 配置)（确保你打开了 docker）

1. 打开浏览器，访问 [http://127.0.0.1:6185](http://127.0.0.1:6185) (如果是部署在云服务器，请把 127.0.0.1 换成你的服务器 IP) 进入 AstrBot 管理后台。
2. 找到模型配置区域，输入你的 **DeepSeek API Key**。
3. 将其设置为“主力模型/全局默认模型”。

### 第四步：唤醒机器人并连接 (NapCat 挂载)（确保你打开了 docker）

1. 浏览器访问 [http://127.0.0.1:6099/webui](http://127.0.0.1:6099/webui) 进入 NapCat 管理页面。
2. 掏出手机，使用准备好的 QQ 小号**扫码登录**。
3. **【最核心的打通操作】**：
   在 NapCat 的 **网络配置** -> **WebSockets 客户端** 选项中，新建一条规则，将 URL 准确填写为 AstrBot 的监听地址：
   ```text
   ws://宿主机IP:6199/ws  
   # (如果是本地同一机器，通常直接填 ws://127.0.0.1:6199/ws 即可)
   ```

---

## 💣 避坑指南 (Troubleshooting)

整理了我在过程中踩过的几个“天坑”。

1. **❓ 坑一：容器全都跑起来了，机器人 QQ 也显示在线，但是发消息就是不回复！**
   * **💡 解决秘籍**：这是网络监听地址没对齐导致的！请务必前往 AstrBot 的机器人配置页（或者直接修改配置 JSON），将反向 WebSocket 监听的主机地址 (`host`) 强制修改为 `0.0.0.0`，同时确保监听端口是 `6199`！改完重启服务，瞬间打通任督二脉！

2. **❓ 坑二：Docker 在 Windows 上死活起不来？**
   * **💡 解决秘籍**：回顾一下环境准备里的踩坑预警 1！请检查你的 Windows 系统的 WSL2 环境是否完备，并且在 Docker Desktop 设置里确认开启了基于 WSL2 的引擎。

3. **❓ 坑三：代码怎么跑不通？环境乱七八糟？**
   * **💡 解决秘籍**：这也是为什么我强烈建议大家使用 Docker！它可以完全隔离你的本地环境，不用再去管那些复杂的 Python 依赖和 Node.js 报错。

---

## 🤝 参与贡献与交流
如果你在部署过程中遇到了新问题，或者有更好的优化建议，欢迎提交 Issue 或 Pull Request！

## 📄 开源协议
本项目采用 [MIT License](LICENSE) 协议开源。
