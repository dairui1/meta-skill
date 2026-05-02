---
name: create-skill
description: 当用户要求创建符合 meta-skill 规范的新 skill 时使用——例如"做一个 terminal-skill"、"给 Notion API 做个 skill"、"我想要一个能自愈的 X 包装层"。生成符合 meta-skill SPEC.md 的骨架（self-heal + 双层 sub-skills + contribute back）。
---

# create-skill

按以下步骤为用户造一个符合 meta-skill 规范的新 skill。**SPEC 是权威，本文件只是流程**。

## Step 0 — 加载 SPEC

fetch `https://github.com/dairui1/meta-skill/raw/main/SPEC.md` 全文先读。下文只是摘要。

## Step 1 — 澄清目标

一次问一个问题：

1. 这个 skill 要操作什么系统？（浏览器、终端、某个 API、桌面应用、文件格式…）
2. 到这个系统**最薄**的直连传输是什么？（CDP websocket、PTY、HTTP、LSP、原生库调用…）
3. 新 skill 放哪里？（默认 `./<name>-skill`）

## Step 2 — 生成骨架

精确创建以下结构，**不多不少**：

```
<name>-skill/
├── README.md            一段说明 + 装到 ~/.claude/skills/ 的提示词
├── SKILL.md             必须提到：先读 helpers / 允许 self-heal /
│                        必须 contribute back / 链接回 meta-skill SPEC
├── helpers.*            一份起步文件，2-3 个跨过所选传输的原语函数；
│                        够做一件事即可，**不要写完整 SDK**
├── interaction-skills/  空目录 + .gitkeep
├── domain-skills/       空目录 + .gitkeep
└── LICENSE              MIT 默认，除非用户指定
```

**绝对不要脚手架**：
- retry / config / logging 框架
- 测试套件
- manager / session / supervisor 层
- 占位 sub-skill 文件（"example.md"、"template.md"）

## Step 3 — 对照 SPEC 自检

重读 SPEC §2「必备结构」和 §5「常见反模式」，遍历刚生成的目录树确认每一条。违反就当场改。

## Step 4 — 首次冒烟

告诉用户**一行**能跑通新 helpers 的命令（例："`echo via PTY`" / "`GET /me on the API`"）。**除非用户要求，自己不跑**——这是用户的权限边界。

## Step 5 — 交接

明确告诉用户：

- skill 已就绪，路径是 `<path>`
- 下一步是用它做真实任务；sub-skills 会随 agent 学习自然累积
- **不要手写 sub-skill 文件**——由 agent 沉淀，人类手写的 skill 不反映真实工作中的有效路径

## 关键纪律

- Step 0 强制远端拉 SPEC：保证本 skill 永远跟规范同步
- Step 2 的 do-NOT 清单：拦住习惯性加框架的冲动
- Step 5 的禁令：人类手写的 skill 不反映真实工作路径，会污染技能库
