# alonelightning-skills

> 开源 AI Agent Skills 合集 —— 都是我们团队日常在用、跑通了一段时间、觉得对别人也有用，才搬出来开源的。

这里的每个 Skill 都是 AI Agent 能直接加载的结构化指令集，遵循 [Agent Skills](https://agentskills.io) 开放标准。Claude Code、Codex、Cursor 等支持该标准的 Agent 都能装。

不追求多，只求真的能干活。

---

## 📦 Skills

| 名字 | 一句话 |
|---|---|
| ☁️ [aliyun-server-guide（阿里云服务器管家）](#-aliyun-server-guide阿里云服务器管家) | 帮零基础用户从零到上线阿里云服务器：选型→购买→部署→域名→备案，代操作 + 手动兜底 |
| 📱 [social-media-content（社媒内容创作工作流）](#-social-media-content社媒内容创作工作流) | 多平台内容创作全链路约束：从选题到出图，核心价值是「约束 > 创意」，让 AI 产出稳定可控 |

---

## ☁️ aliyun-server-guide（阿里云服务器管家）

> 不是又一篇购买教程，而是一个能让 AI agent **真正替你动手操作**的技能包。

普通人想上线一个网站，卡在一堆门槛上：不会选配置、看不懂术语、不敢碰命令行、备案一头雾水。这个 skill 把 agent 定位成**代操作管家**：

> **能自动做的自动做；做不了的给链接手把手教；每个自动步骤都有手动兜底；每一步都说清在干什么；花钱和实名的事必须先问过你。**

**真能自动**：ECS 询价下单（RunInstances）、免 SSH 部署（云助手 RunCommand）、DNS 解析、HTTPS 证书签发
**诚实标注做不了**：注册 / 实名（人脸）/ AccessKey 创建 / 活动价下单 / ICP 备案（阿里云无 OpenAPI，全程陪跑指引）

**关键设计**：透明播报 · 花钱等 7 类关键动作执行前必须获得你明确同意（花钱必先报价）· 自动失败最多重试 2 次转手动兜底 · 无 AccessKey 则退化为纯指引模式 · 凭证安全红线（RAM 最小权限 AK、Secret 不落聊天记录）

**怎么触发**：`帮我买阿里云服务器` / `云服务器怎么选` / `帮我部署网站到服务器` / `域名解析` / `ICP 备案`

→ [SKILL.md](/alonelightning/alonelightning-skills/blob/main/aliyun-server-guide/SKILL.md)

> ⚠️ 政策与价格随阿里云调整，最终以阿里云控制台当前要求为准。本 skill 与阿里云官方无任何关联。

---

## 📱 social-media-content（社媒内容创作工作流）

> 不是教你「怎么写爆款」，而是一套让 AI 内容产出**稳定可控**的工程化约束。

核心理念一句话：**Skill 的价值在于约束，不在于创意。** 工具解决「能做什么」，这个 skill 解决「什么时候做、怎么做、做到什么标准」。覆盖小红书 / 视频号 / 公众号多平台，从选题、调研、写稿、审核到配图出图的全链路。

**里面有什么**：
- **13 步标准流程 + 阻断点**：每一步谁做、卡在哪要停下来问人
- **配图 Prompt 四段式标准结构**：让 AI 出图时把中文文字准确渲染进图里（信息入图，正文只留钩子）
- **🔴 两条最高红线**：来源归属（内容来自别人必须点名到人/出处，不点名=据为己有）+ 素材铁律（必带可核验原文链接，禁止凭记忆编来源）
- **协作模式**：多 Agent 协作时「选题对齐 → 执行全权」，单篇单会话防止多篇互相污染致幻
- **出图执行规范**：基于 Codex CLI 批量生图，含文本渲染质量检查

**怎么触发**：`帮我写小红书文案` / `多平台内容分发` / `配图关键词` / `选题讨论`

→ [SKILL.md](/alonelightning/alonelightning-skills/blob/main/social-media-content/SKILL.md) · [完整指南](/alonelightning/alonelightning-skills/blob/main/social-media-content/%E5%AE%8C%E6%95%B4%E6%8C%87%E5%8D%97.md)

---

## 🔧 怎么安装

在 Claude Code、Codex 等支持 Agent Skills 的工具里，直接说：

```
帮我安装这个 skill：https://github.com/alonelightning/alonelightning-skills/tree/main/aliyun-server-guide
```

Agent 会自己 clone 到对应目录。

不支持 Skill 的 Agent 也没关系：把对应目录的 `SKILL.md` 全文下载下来，当成项目规则文件（或直接贴进对话）让 Agent 照着执行，效果一致。

---

## 关于我们

一个人，和一群有名字的 AI 伙伴，组成的「人 + AI」协作团队。把 AI 当作伙伴而非工具——所以给每个伙伴都起了名字。

这些 skill 都是我们协作中沉淀下来的东西。开源出来如果对你有帮助，给个 ⭐ 就行。有问题或建议，欢迎在 Issues / Discussions 里说一声。

## License

[MIT](https://github.com/alonelightning/alonelightning-skills/blob/main/LICENSE)
