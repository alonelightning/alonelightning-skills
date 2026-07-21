# Codex CLI 首次使用完整指南

本指南面向在 OpenClaw / EasyClaw（或其他基于 OpenClaw 的封装版本）中**首次调用 Codex CLI** 的用户。即使你的平台已经内置了 Codex CLI，首次使用时仍然会遇到一系列需要手动处理的问题。

---

## 前置条件

| 条件 | 说明 |
|------|------|
| OpenClaw 或基于 OpenClaw 的平台 | EasyClaw、或其他封装版本均可 |
| OpenAI / ChatGPT 账户 | Codex CLI 通过 ChatGPT 账户认证，不需要 API Key |
| 系统代理（VPN） | 在中国大陆必须开启，否则无法连接 OpenAI 服务器 |
| Node.js v18+ | Codex CLI 依赖 Node.js 运行 |

---

## 第一步：确认 Codex CLI 是否已安装

在你的平台终端（或通过 Agent 的 exec 工具）运行：

```powershell
codex --version
```

### 如果输出版本号（如 `v0.130.0`）
已安装，跳到第二步。

### 如果提示"未找到命令"
**可能情况 A**：平台内置了但没加到 PATH。

EasyClaw 内置路径通常在：
```
%LOCALAPPDATA%\easyclaw\ai\tool_cache\resources\tools\win\node-{版本}\codex.cmd
```

用完整路径调用：
```powershell
& "$env:LOCALAPPDATA\easyclaw\ai\tool_cache\resources\tools\win\node-24.13.0\codex.cmd" --version
```

找到后可以继续用完整路径，或将该目录添加到系统 PATH。

**可能情况 B**：平台没有内置 Codex CLI。

手动安装：
```powershell
# 确认 Node.js 已安装
node --version

# 如果没有 Node.js
winget install OpenJS.NodeJS.LTS    # Windows
# 或 brew install node               # macOS

# 全局安装 Codex CLI
npm install -g @openai/codex

# 验证
codex --version
```

---

## 第二步：登录 ChatGPT 账户

**⚠️ 这是首次使用必须做的一步，不登录就无法调用任何模型。**

```powershell
codex login
```

浏览器会弹出 OpenAI 登录页面，使用你的 ChatGPT 账户登录授权。登录成功后凭证自动保存在本地。

### 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 浏览器没弹出 | 系统代理未开启 | 先开代理再 login |
| 登录后仍提示未认证 | 凭证缓存异常 | 删除 `~/.codex/` 目录后重新 login |
| EasyClaw 内 Agent 调用时无法弹浏览器 | Agent 在后台运行 | 在 PowerShell 中手动执行 `codex login` 完成登录后，Agent 可复用凭证 |

---

## 第三步：确认网络环境

**⚠️ 这是最容易出问题的环节，也是导致首次调用失败最常见的原因。**

### 检查清单

- [ ] 系统代理（VPN/梯子）已开启
- [ ] 浏览器能访问 `https://api.openai.com`
- [ ] 浏览器能访问 `https://chatgpt.com`

### 快速验证

```powershell
# 简单测试 Codex 能否连通
codex exec -m gpt-5.5 "Say hello" 2>&1
```

如果看到正常回复 → 网络正常。
如果看到 `Reconnecting... 1/5` → 代理没开或节点不通。

---

## 第四步：首次图片生成测试

```powershell
# 创建测试目录
mkdir codex-test
cd codex-test

# 测试生成一张图片
codex exec -m gpt-5.5 -s danger-full-access "Generate a simple 800x800 test image with the text 'Hello World' on a blue gradient background. Save as test.png in the current directory." 2>&1
```

### 预期流程

1. 先看到 `Reading additional input from stdin...`（正常）
2. 可能看到 `failed to refresh available models: timeout`（忽略，不影响执行）
3. 等待 30-60 秒
4. 看到任务完成，`test.png` 出现在当前目录

### 如果失败了

按以下顺序排查：

| 症状 | 排查方向 |
|------|---------|
| `Reconnecting... 5/5` → 进程退出 | 代理没开，或代理节点连不上 OpenAI |
| `failed to refresh models` 后无后续输出 | 等一等（可能在加载），超过 2 分钟再考虑重试 |
| `Transport channel closed` | 网络波动，直接重试 |
| 运行成功但图片找不到 | 检查 `~/.codex/generated_images/` 是否有图片，prompt 中是否写了保存指令 |
| `-m gpt-image-2` 连接失败 | **这是最常见的新手错误**：gpt-image-2 不是聊天模型，改用 `-m gpt-5.5` |

---

## 第五步：正式使用（配图生成标准流程）

通过测试后，按以下标准流程生成配图：

```powershell
# 1. 进入目标目录
cd "C:\workspace\xhs-images\1a"

# 2. 执行生成命令
codex exec -m gpt-5.5 -s danger-full-access "我要做一张用来发布在社交媒体的图片，1:1的宽高比，像素在1600*1600，非专有名词和单位的文案变成中文：[英文设计指令]。Generate and save as cover.png in the current directory." 2>&1
```

### 关键参数说明

| 参数 | 含义 | 是否必须 |
|------|------|---------|
| `exec` | 单次执行模式 | 必须 |
| `-m gpt-5.5` | 聊天模型（会自动调用 image_gen） | 必须，可换其他聊天模型 |
| `-s danger-full-access` | 允许文件写入 | 生成图片时必须 |
| `2>&1` | 合并 stderr 到 stdout | 推荐（方便 Agent 捕获完整输出） |

### 并行生成

可同时启动多个进程：
```powershell
# 在 OpenClaw/EasyClaw 中通过 exec + background 启动多个后台进程
# 通过 process poll 检查状态
# ⚠️ 轮询间隔 ≥ 10-15 秒，单张图片通常需要 30-60 秒
```

---

## 版本兼容说明

| Codex CLI 版本 | 状态 | 备注 |
|----------------|------|------|
| v0.130.0+ | ✅ 验证通过 | 推荐 |
| v0.120.x | ⚠️ 未测试 | 可能可用，建议升级 |
| < v0.120 | ❌ 不推荐 | 可能缺少 image_gen 工具支持 |

升级命令：
```powershell
npm update -g @openai/codex
```

---

## 新环境部署一键脚本（Windows PowerShell）

以下脚本帮你在一台新电脑上快速配置好 Codex CLI 环境：

```powershell
# === Codex CLI 一键部署脚本 ===
# 前置条件：Node.js v18+ 已安装、系统代理已开启

# 1. 安装 Codex CLI
npm install -g @openai/codex

# 2. 验证安装
$version = codex --version
Write-Host "Codex CLI installed: $version"

# 3. 提示登录
Write-Host ""
Write-Host "=== 请执行以下命令完成登录 ==="
Write-Host "codex login"
Write-Host ""
Write-Host "登录成功后，运行以下命令测试："
Write-Host 'codex exec -m gpt-5.5 "Say hello" 2>&1'
```

---

## 排错速查卡

| 症状 | 原因 | 一句话解决 |
|------|------|-----------|
| `Reconnecting... 5/5` | 代理没开 | 开 VPN |
| `-m gpt-image-2` 连接失败 | 模型选错 | 改 `-m gpt-5.5` |
| `timeout refresh models` | 正常现象 | 忽略 |
| `transport channel closed` | 网络抖动 | 重试 |
| 图片找不到 | 没指定路径 | prompt 加 "save as X.png" |
| 未认证 | 没登录 | `codex login` |
| 浏览器没弹出 | 代理没开 | 先开代理再 login |
| Agent 调用时无法登录 | 后台运行 | 手动在终端 `codex login` |
