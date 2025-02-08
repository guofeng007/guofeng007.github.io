---
layout: post
title: 免费本地部署！手把手教你离线运行国产大模型DeepSeek
categories: [Blog,Cat]
description: gpt deepseek aigc
keywords: gpt deepseek aigc
---

### 🔥免费本地部署！手把手教你离线运行国产大模型DeepSeek

---

📢 **无需联网！无需付费！** 今天教你在个人电脑上部署最新国产大模型DeepSeek，搭配可视化界面实现媲美ChatGPT的智能问答体验，全程只需10分钟！

---

## 🛠️ 准备工作

1️⃣ 硬件要求：

- 最低配置：8GB内存+4核CPU（1.5b版本）
- 推荐配置：16GB内存+独立显卡（7b/8b版本）
- 本人亲测mac m3 16G+256G配置

2️⃣ 软件准备：

- 下载Ollama：[https://ollama.com](https://ollama.com)
- 下载Chatbox客户端：[Chatbox AI官网：办公学习的AI好助手，全平台AI客户端，官方免费下载](https://chatboxai.app/zh/)

---

## 🚀 四步极速部署指南

### 第一步：安装Ollama服务

- **Windows用户**：双击安装包后，右下角会出现🦙小羊驼图标

- **Mac用户**：拖动到Applications文件夹后，顶部菜单栏可见图标

- **终端验证安装**：
  
  ```bash
  ollama --version
  ```

### 第二步：拉取DeepSeek模型

根据设备性能选择模型（👇性能对比）：

```bash
# 低配设备推荐（1.5b约占用2GB内存）

ollama run deepseek-r1:1.5b

# 高配设备可选（7b约需8GB内存）

ollama run deepseek-r1:7b
```

💡 **小技巧**：输入`ollama list`可查看已下载模型

### 第三步：配置Chatbox客户端

1. 打开Chatbox点击⚙️设置

2. 选择模型提供方 → **Ollama API**

3. 填写配置：
   
   ```bask
   API地址：http://127.0.0.1:11434
   模型名称：deepseek-r1:7b
   ```

4. 点击「保存」即完成连接

### 第四步：离线问答测试

尝试输入：

```bash
请用鲁迅的风格写一段关于AI的评论
```

💻 此时所有计算都在本地运行，断网也能正常使用！

---

## 🎯 进阶玩法

✨ **企业级应用**：通过AnythingLLM接入公司知识库  
✨ **角色扮演**：设置AI为「毒舌吐槽君」或「温柔心理师」  
✨ **远程访问**：修改Ollama配置实现跨设备调用

---

## ⚠️ 常见问题排雷

🔸 **模型加载失败** → 检查Ollama服务是否运行  
🔸 **响应速度慢** → 切换低参数量版本或关闭其他程序  
🔸 **端口冲突** → 修改启动命令指定端口：

```bash
OLLAMA_HOST=0.0.0.0:54321 ollama serve
```

---

## 📈 性能实测对比

| 模型版本 | 内存占用    | 生成速度（字/秒） | 适合场景      |
| ---- | ------- | --------- | --------- |
| 1.5b | 2-3GB   | 45        | 文案生成/基础问答 |
| 7b   | 8-10GB  | 28        | 代码编写/逻辑推理 |
| 8b   | 12-14GB | 18        | 专业领域分析    |

---

💡 **彩蛋功能**：在Chatbox输入`/角色卡`可快速切换AI人格，已内置20+种预设角色！

立即动手部署，体验属于你的私人AI助手吧！更多技巧欢迎在评论区交流讨论👇