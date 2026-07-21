# Codex CLI 图片生成完整指南

## 概念说明

Codex CLI 是 OpenAI 提供的命令行工具，可以在终端中直接调用 GPT 系列模型。当用聊天模型（如 gpt-5.5）执行图片生成任务时，模型内部会自动调用 `image_gen` 工具（底层为 gpt-image-2），无需手动指定图片模型。

**关键理解**：Codex CLI 的 `-m` 参数只接受**聊天模型**（能对话的模型），不接受纯功能模型。图片生成是聊天模型的一个"技能"，不是独立的模型端点。

## 正确调用方式（⭐ 写文件再读取）

> **核心原则**：禁止在命令行或 stdinData 中直接传递中文/emoji prompt。必须先写文件再让 codex 读取。

### 步骤

1. **写 prompt 文件**：将完整 prompt 写入产出目录下的 `.md` 文件
   - 文件名：`prompt-<图片名不带扩展名>.md`（如 `prompt-cover.md`）
   - 编码：UTF-8，中文/emoji/特殊符号零损耗
   - 末尾加一行：`Generate image with size 1024x1536 and save as <英文文件名>.png`

2. **codex 读文件生图**：
   ```bash
   codex exec -m gpt-5.5 -s danger-full-access --skip-git-repo-check -C "<产出目录>" "读取当前目录下的 prompt-<name>.md 文件，用其内容作为 prompt 调用 image_gen 工具生成图片，保存为 <name>.png"
   ```

3. **完成后**：删除 prompt-*.md 中间文件

### 为什么必须写文件

| 方式 | 问题 |
|------|------|
| 命令行直接拼中文 | PowerShell 编码转换导致乱码/codex 重做 |
| stdinData 传中文 | 编码不可控，emoji 和特殊符号容易丢失 |
| 写 .md 文件让 codex 读 | ✅ UTF-8 编码，零损耗，稳定可靠 |

### 参数解释

| 参数 | 含义 |
|------|------|
| `exec` | 单次执行模式（非交互式） |
| `-m gpt-5.5` | 指定聊天模型（必须是 gpt-5.5，不得降级） |
| `-s danger-full-access` | 安全等级，允许文件读写 |
| `--skip-git-repo-check` | 跳过 Git 仓库检查（工作目录不在 Git 内时必须加） |
| `-C <目录>` | 指定工作目录 |

## 并行生成

- 一次发起 **≤5 个**并行 codex 进程（background 模式）
- 禁止单张串行（单张约 5-8 分钟，串行 11 张 = 70+ 分钟）
- 5 路并行：11 张约 2-3 批，15-20 分钟完成
- 发完一批后用一次长等待（≥8 分钟 timeout），不逐张 poll

## 工作目录隔离

codex 在 `danger-full-access` 下会自发探索 cwd 周边文件。**出图时把工作目录指到纯出图目录**（只有 prompt/图），避免读取敏感文件浪费 token。

## 质量检查

### 文件大小判定
- **>1MB** = 真 AI 生成（gpt-image-2），✅ 通过
- **<500KB** = 大概率 Pillow/代码画的，❌ 重做

### 文本渲染检查（⭐ 新增）
出图全部完成后，用视觉模型（如 image 工具）逐张检查：
- 图上文字是否与 Prompt 中的文案一致
- 检查项：错字、漏字、多字、排版严重错乱、文字被截断/遮挡
- 严重问题 → 重试 1 次
- 轻微变形但可识别 → 标记"警告"，由协调者决定是否重做

### 尺寸
- 标准：1080×1920（9:16）
- 生成参数：`size 1024x1536`（codex 实际输出会接近目标比例）

## 文件组织规范

```
产出目录/
├── 2026-06-25/
│   ├── topic-name-1/
│   │   ├── 文案.md
│   │   ├── cover.png
│   │   ├── data-comparison.png
│   │   └── features.png
│   ├── topic-name-2/
│   │   ├── 文案.md
│   │   ├── cover.png
│   │   └── ...
```

## 踩坑记录与排错

### ❌ 中文文件名

```
codex exec "... 保存为 配图1-封面.png ..."  → 编码解析失败，反复重试
```

**正解**：文件名统一用英文。prompt 中的中文描述保留在 .md 文件里。

### ❌ Codex fallback 到 Pillow/PIL

**现象**：Codex 不调用 image_gen，写 Python 代码用 Pillow 画图。文件小（<500KB），质量极低。
**原因**：网络不稳定或 Reconnecting 失败后 Codex 自动切换策略。
**防护**：prompt 开头加：`Use the image generation tool to create this image, do NOT write code or use Pillow/PIL.`

### ❌ `-m gpt-image-2`

**原因**：gpt-image-2 不是聊天模型，Codex CLI 的 `-m` 只接受聊天模型。
**正解**：使用 `-m gpt-5.5`。
**约束**：永远不要把 gpt-image-2 作为 `-m` 参数。

### ⚠️ `failed to refresh available models: timeout`

WARNING 级别，不影响执行，忽略即可。

### ⚠️ `Reconnecting... 1/5 ~ 5/5`

**原因**：系统代理（VPN）未开启或网络波动。
**正解**：确保系统代理已开启。

### ⚠️ `rmcp transport worker quit`

网络不稳定导致 WebSocket 断开，直接重试。

### 📁 图片保存路径

Codex 默认把图片存在 `~/.codex/generated_images/`。
**约束**：prompt 末尾必须明确指定保存文件名（`save as <name>.png`），并通过 `-C` 指定工作目录。

### 🔑 首次使用需要登录

```bash
codex login  # 浏览器弹出 OpenAI 登录页
```

## 版本兼容

| Codex CLI 版本 | image_gen 支持 | 建议 |
|----------------|---------------|------|
| v0.13.0+ | ✅ 推荐 | 稳定性最好 |
| v0.9.0+ | ✅ 可用 | — |
| < v0.9.0 | ❌ 不支持 | 必须升级 |

升级：`npm update -g @openai/codex`

---

**版本**：2.0.0 | **更新**：2026-06-25
