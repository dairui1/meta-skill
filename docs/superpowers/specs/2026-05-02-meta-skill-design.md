# meta-skill 设计文档

**日期**：2026-05-02
**作者**：dairui1 + Claude
**仓库**：https://github.com/dairui1/meta-skill
**状态**：设计已确认，待生成实施计划

---

## 1. 背景与目标

`browser-use/browser-harness` 用极薄的代码（~600 行 Python）+ 一份 SKILL.md，让 LLM 可以自由操作浏览器：当 helpers 缺功能时 agent 直接改源文件（self-heal/自愈），学到非显然的东西沉淀进 `interaction-skills/` 和 `domain-skills/` 两层目录。这套范式不是浏览器专属——它对任何"agent 长期操作某个外部系统"的场景都成立。

**meta-skill 的目标**：把 browser-harness 这套范式抽象成一份**通用规范**，让任何能力领域（终端、API、文件系统、设计工具、IDE…）都能照着长出符合同一精神的 skill。

**非目标**：
- 不做脚手架仓库（不附带可 clone 的模板代码）
- 不附带参考实现（browser-harness 自己就是活样板）
- 不强制传输层、不强制语言、不强制总代码量

**术语澄清**：本文中"skill"指**符合 meta-skill 规范的 agent 能力包**——它是一个目录，含 `SKILL.md` + `helpers.*` + 双层 sub-skills 目录。browser-harness 本身就是一个这样的 skill。

---

## 2. 核心精神（三条不可破）

只保留 browser-harness 范式中真正可移植的三条原则。其它（薄、单一直连协议、代码即文档）作为推荐而非强制。

### 2.1 Self-heal（自愈）
helpers 不全时，agent **直接编辑** helpers 源文件补齐功能，而不是用更上层的抽象绕过。helpers 必须是 agent 可读可改的源代码——不允许是只读 SDK 黑盒。

### 2.2 双层 sub-skills
所有沉淀放进两个固定目录：
- `interaction-skills/`：与目标系统交互时的**通用机制**（怎么做）。例：浏览器的 dialog/iframe/upload；终端的 PTY/ANSI/信号；API 的 auth/分页/重试。
- `domain-skills/`：针对**具体目标**的工作流（对谁做）。例：浏览器的 tiktok/linkedin；API 的 notion/stripe；文件系统的 docker/git。

命名沿用 browser-harness 原词，不抽象成 `mechanic-skills/` 或 `pattern-skills/`——这两个词本就够通用，沿用换来生态复利。

### 2.3 Contribute back（沉淀回传）
agent 学到非显然的事必须沉淀成 sub-skill 文件。值得沉淀的：私有 API、稳定 selector / 命令、框架怪癖、URL 或路径模式、wait 的原因、陷阱。**不该写**的：像素坐标、任务流水账、密钥/会话状态。

---

## 3. 产物形态

### 3.1 仓库本体（`dairui1/meta-skill`）

```
meta-skill/
├── README.md           ← 一段话定义 + 安装提示词
├── SPEC.md             ← 规范本体（中文）
├── skill/
│   └── SKILL.md        ← create-skill skill 源文件（中文）
└── LICENSE
```

**单一信源**：`SPEC.md` 是真理之源（什么是合格的 skill）；`skill/SKILL.md` 是流程之源（怎么造一个），通过远端拉取 SPEC，自己不复述规范内容。

### 3.2 用户级 skill（安装后落到本地）

```
~/.claude/skills/create-skill/
└── SKILL.md            ← 来自仓库 skill/SKILL.md
```

不在仓库里附带、由用户在 Claude Code 里跑安装提示词后由 agent 自己写入。

---

## 4. SPEC.md 章节大纲

总长目标 ~1000 字以内，一屏可读完。

| # | 章节 | 字数 | 内容 |
|---|---|---|---|
| 1 | 这是什么 | ~80 | 一句话定义 skill / meta-skill，举 browser-harness |
| 2 | 三条不可破原则 | ~300 | 2.1 Self-heal / 2.2 双层 sub-skills / 2.3 Contribute back |
| 3 | 必备结构 | ~200 | 最小合格 skill 的目录树 + 必须存在的文件 |
| 4 | Self-heal 协议 | ~150 | agent 在 helpers 不全时的 4 步标准动作 |
| 5 | Contribute-back 规则 | ~120 | 什么算"值得沉淀" vs "不该写进 sub-skill" |
| 6 | 参考实现 | ~30 | 一行链接到 browser-harness |
| 7 | 常见反模式 | ~150 | retry framework、helpers 变 SDK、sub-skills 变静态文档… |
| 8 | License & contributing | ~30 | |

---

## 5. create-skill 的 SKILL.md 章节大纲

```
---
name: create-skill
description: 当用户要求创建符合 meta-skill 规范的新 skill 时使用——例如
  "做一个 terminal-skill"、"给 Notion API 做个 skill"、"我想要一个能自愈的
  X 包装层"。生成符合 meta-skill SPEC.md 的骨架（self-heal + 双层 sub-skills
  + contribute back）。
---

# create-skill

## Step 0 — 加载 SPEC
fetch https://github.com/dairui1/meta-skill/raw/main/SPEC.md 全文先读。
下面只是摘要，SPEC 是权威。

## Step 1 — 澄清目标（一次一个问题）
1.1 这个 skill 要操作什么系统？
1.2 到这个系统最薄的直连传输是什么？
1.3 新 skill 放哪里？（默认 ./<name>-skill）

## Step 2 — 生成骨架
精确创建：
  <name>-skill/
    README.md       (一段 + 安装提示词，仿 browser-harness)
    SKILL.md        (必须提到：先读 helpers / 允许 self-heal / 必须 contribute back / 链接 SPEC)
    helpers.*       (一份起步文件，2-3 个跨过所选传输的原语函数；够做事即可，别写完整 SDK)
    interaction-skills/   (空目录 + .gitkeep)
    domain-skills/        (空目录 + .gitkeep)
    LICENSE

不要脚手架：
  - retry / config / logging 框架
  - 测试套件
  - manager / session / supervisor 层
  - 占位 sub-skill 文件（"example.md"）

## Step 3 — 对照 SPEC 自检
重读 SPEC「必备结构」「常见反模式」，遍历生成的目录树确认每条要求。违反就当场改。

## Step 4 — 首次冒烟
告诉用户一行能跑通新 helpers 的命令。除非用户要求，自己不跑。

## Step 5 — 交接
告诉用户：
  - skill 已就绪，路径是 <path>
  - 下一步是用它做真实任务；sub-skills 会随 agent 学习自然累积
  - 不要手写 sub-skill 文件——由 agent 沉淀
```

**关键设计点**：
- **Step 0 强制远端拉 SPEC**——保证 skill 永远和规范同步
- **Step 2 列 "do NOT scaffold" 清单**——拦住 agent 习惯性加框架的冲动
- **Step 5 禁止人类手写 sub-skill**——这是 browser-harness README 的原话精神

---

## 6. 分发与安装

### 6.1 README.md 内容

```markdown
# meta-skill

一份规范，用来造"薄、能自愈、技能能沉淀"的 agent skill。
范式源自 browser-use/browser-harness，本仓库把它抽象成通用形态。

→ 请读 SPEC.md。

## 安装 create-skill skill

把下面这段贴进 Claude Code：

  请读 https://github.com/dairui1/meta-skill/raw/main/SPEC.md 理解
  meta-skill 范式。然后把 https://github.com/dairui1/meta-skill/raw/
  main/skill/SKILL.md 的内容下载，写入 ~/.claude/skills/create-skill/
  SKILL.md。完成后确认文件存在，并告诉我以后可以随时让你
  "做一个 <名字>-skill"。

零依赖、不用 clone、不用装脚本。
```

### 6.2 设计取舍

- **skill 放 `skill/` 子目录**（而非仓库根 `SKILL.md`）：避免 Claude Code 把它误识别为本仓库自身的 skill。
- **升级路径**：用户重跑安装提示词即可覆盖；SPEC 由 skill 在 Step 0 现拉，无本地副本，自动跟随 main。
- **不做 install.sh 或 plugin**：一段提示词 = 零依赖、跨平台、agent 自己执行——meta-skill 自身也践行"薄"。

---

## 7. 语言

- `SPEC.md`、`skill/SKILL.md`、`README.md` **全部中文**。
- 文件/目录名保留英文（`helpers.py`、`interaction-skills/` 等）。
- 行话术语首次出现时给中文释义（如 `self-heal` → 自愈）。

---

## 8. 常见反模式（写进 SPEC 第 7 节）

写在这里供实施时直接拷贝：

1. 给 helpers 加 retry / circuit-breaker / config 系统
2. 把 helpers 变成只读 SDK，agent 不许改
3. 让人类手写 sub-skill 文件，而不是让 agent 沉淀
4. 把任务流水账（"今天我做了 ABC"）当 sub-skill 提交
5. 在 sub-skill 里写像素坐标、cookie、token、用户名
6. 为"未来可能的需求"提前抽象（违反 YAGNI）
7. 加 manager / supervisor / session-pool 层
8. 把 SPEC 内容复制进 skill（应当现拉，避免漂移）

---

## 9. 后续步骤

- [ ] 由用户审阅本设计文档
- [ ] 通过后调用 `writing-plans` skill 生成实施计划
- [ ] 实施计划应覆盖：
  - 在 `dairui1/meta-skill` 仓库写 SPEC.md / skill/SKILL.md / README.md / LICENSE
  - 首次端到端验证：用 `create-skill` skill 自己造一个 toy skill 验证流程
  - commit & push 到 GitHub
