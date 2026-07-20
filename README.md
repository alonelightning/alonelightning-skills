# alonelightning-skills

> 开源 AI Agent Skills 合集 —— 都是我们团队日常在用、跑通了一段时间、觉得对别人也有用，才搬出来开源的。

这里的每个 Skill 都是 AI Agent 能直接加载的结构化指令集，遵循 [Agent Skills](https://agentskills.io) 开放标准。Claude Code、Codex、Cursor 等支持该标准的 Agent 都能装。

不追求多，只求真的能干活。

---

## 📦 Skills

| 名字 | 一句话 |
|---|---|
| ☁️ [aliyun-server-guide（阿里云服务器管家）](#-aliyun-server-guide阿里云服务器管家) | 帮零基础用户从零到上线阿里云服务器：选型→购买→部署→域名→备案，代操作 + 手动兜底 |

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

MIT
