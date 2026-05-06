---
name: create-harness
description: 当用户要求创建 harness 形态的 skill 时使用——例如"做一个 seo-harness"、"给终端做个 test-harness"、"我想要 browser-harness 那样的能自愈包装层"。生成符合 meta-skill SPEC.md 的 harness 骨架（CLI 在 $PATH + agent-workspace 分离 + install.md + 富 SKILL.md）。
---

# create-harness

为用户造一个 **harness 形态**的 skill：薄核心 + agent 可改的 workspace + 单命令 CLI + 自愈协议。**SPEC 是权威，本文件只是流程**。

harness 不同于最小 skill：它假设 agent 会反复调用一个长生命周期的工具（浏览器、终端、爬虫、测试 runner …），所以多了 CLI 入口、`agent-workspace/`、`install.md`、富 SKILL.md。如果用户只要一个轻量 API 包装，告诉他 harness 是过度形态，建议换最小 skill 即可。

## Step 0 — 加载 SPEC

fetch `https://github.com/dairui1/meta-skill/raw/main/SPEC.md` 全文先读。harness 是 SPEC §2 的一种**加厚实例**，三条原则照旧。

## Step 1 — 澄清目标

一次问一个问题：

1. 操作什么系统？（浏览器、终端 PTY、SEO/爬虫栈、测试 runner、桌面应用 …）
2. 最薄的直连传输是什么？（CDP WS、PTY、HTTP、子进程 stdout、原生库 …）
3. 需要长驻 daemon 吗？（连接型 / 有状态会话 → 是；一次性 HTTP 调用 → 否）
4. 语言？（默认 Python，因为本范式起点是 browser-harness；其他语言把 pyproject/src 换成对应包管理即可）
5. 新 harness 放哪里？（默认 `./<name>-harness`）
6. CLI 命令名？（默认 `<name>-harness`）

## Step 2 — 生成骨架

精确创建以下结构，**不多不少**（以下为 Python 默认；非 Python 时把 `pyproject.toml`/`src/` 换成对应包布局，其他不变）：

```
<name>-harness/
├── README.md                       一段说明 + 安装提示词
├── SKILL.md                        入口（给「使用 harness」的 agent）；含 Usage /
│                                   What actually works(空) / Gotchas(空) /
│                                   Design constraints / Contribute back / 链接回 SPEC
├── AGENTS.md                       入口（给「修改 harness」的 agent）；3-5 行：
│                                   代码优先级 + 哪些目录是 core、哪些是 workspace +
│                                   「使用 harness 的 agent 只编辑 agent-workspace/」
├── install.md                      装到 $PATH + 接入 agent + 冷启动/重连调试；
│                                   预留 --setup / --doctor / --update 生命周期 flag
├── pyproject.toml                  含 [project.scripts] <name>-harness = "<pkg>.run:main"
├── src/<pkg_name>/
│   ├── __init__.py                 空
│   ├── run.py                      CLI 入口；解析 -c '<code>'，exec 时把 helpers 预导入
│   ├── helpers.py                  2-3 个跨过传输的原语；够做一件事即可
│   └── daemon.py                   **仅在 Step 1.3 选了 daemon 时**；否则不创建
├── agent-workspace/
│   ├── agent_helpers.py            空文件 + 一行注释「使用 harness 的 agent 只在这里加函数；
│   │                               不要动 src/」
│   └── domain-skills/.gitkeep
├── interaction-skills/.gitkeep
└── LICENSE                         MIT 默认
```

**SKILL.md 必须含的章节**（按 browser-harness 模式）：
- frontmatter：`name`（**用短名词，不带 `-harness` 后缀**——例如 `name: browser`、`name: seo`，让 agent 端调用更自然）、`description`（描述触发场景）
- Usage：一行 `<name>-harness -c '...'` 范例
- What actually works：留空 + 注释「agent 沉淀」
- Gotchas：留空 + 注释「agent 沉淀」
- Design constraints：写死本 harness 的硬约束（不加 retry / 不加 manager / 不启动自己的进程 / core 保持薄 …）
- Always contribute back：复述 SPEC §4 的"写什么/不写什么"
- Search first：提示先 grep `agent-workspace/domain-skills/` 和 `interaction-skills/`

**绝对不要脚手架**：
- retry / config / logging / circuit-breaker 框架
- 测试套件（tests/ 目录留空都不行——别建）
- manager / session-pool / supervisor 层
- 占位 sub-skill 文件（example.md / template.md / TODO.md）
- 把 SPEC 内容复制进 SKILL.md（应当链接，避免漂移）

**run.py 起步形状**（Python 例）：把 helpers 全部预导入，`-c` 后面的代码直接 `exec` 到那个命名空间。**不要** argparse 子命令、不要 click 装饰器、不要把 helpers 包成 class。

## Step 3 — 对照 SPEC 自检

重读 SPEC §2 与 §5，再额外查 harness 不变量：

- [ ] CLI 装好后能在任意目录直接 `<name>-harness -c '...'`，无需 cd / uv run
- [ ] core (`src/`) 没有 retry / manager / config 系统
- [ ] 使用 harness 的 agent 加新函数会落到 `agent-workspace/agent_helpers.py`，不动 `src/`
- [ ] `interaction-skills/` 与 `agent-workspace/domain-skills/` 都存在且为空
- [ ] SKILL.md 与 AGENTS.md 各司其职——前者给「使用」的 agent，后者给「修改」的 agent
- [ ] SKILL.md 链接了 SPEC，没有复制 SPEC 内容
- [ ] 没有写测试套件、没有占位 sub-skill

违反就当场改。

## Step 4 — 首次冒烟

告诉用户**两条**命令：

1. 安装：`cd <path> && uv tool install -e .`（或对应语言的等价命令）
2. 验证：`<name>-harness -c 'print("ok")'`

**除非用户要求，自己不跑**——这是用户的权限边界。

## Step 5 — 交接

明确告诉用户：

- harness 已就绪，路径 `<path>`，CLI `<name>-harness`
- 下一步是用它做真实任务；`agent-workspace/agent_helpers.py` 和两个 sub-skills 目录会随 agent 学习自然累积
- **不要手写 sub-skill 文件**——由 agent 沉淀，人类手写的不反映真实有效路径
- 任务时 self-heal 默认落 `agent-workspace/agent_helpers.py`；只有当缺的是**跨任务通用的传输原语**才进 `src/<pkg>/helpers.py`

## 关键纪律

- Step 0 强制远端拉 SPEC：保证本 skill 永远跟规范同步
- Step 2 的 do-NOT 清单：拦住"加点框架更专业"的冲动；harness 的价值在薄
- Step 2 区分 `src/` 与 `agent-workspace/`：core 稳定、workspace agent 可改，是 harness 模式的核心分界
- Step 5 的禁令：人类手写的 skill 不反映真实工作路径，会污染技能库
