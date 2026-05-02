# meta-skill SPEC

一份规范，定义"什么是合格的 skill"——薄、能自愈（self-heal）、技能能沉淀。可在任何能力领域复用：浏览器、终端、API、设计工具…

本文中"skill"指**符合本规范的 agent 能力包**：一个目录，含 `SKILL.md` + `helpers.*` + 双层 sub-skills 目录。"meta-skill"指本规范本身。

---

## 1. 三条不可破原则

### 1.1 Self-heal（自愈）
helpers 不全时，agent **直接编辑** helpers 源文件补齐功能，而不是用更上层的抽象绕过。helpers 必须是 agent 可读可改的源代码——不允许是只读 SDK 黑盒。

### 1.2 双层 sub-skills
所有沉淀放进两个固定目录：
- `interaction-skills/`：与目标系统交互的**通用机制**（怎么做）。例：浏览器的 dialog/iframe/upload；终端的 PTY/ANSI/信号；API 的 auth/分页/重试。
- `domain-skills/`：针对**具体目标**的工作流（对谁做）。例：浏览器的 tiktok/linkedin；API 的 notion/stripe；文件系统的 docker/git。

### 1.3 Contribute back（沉淀回传）
agent 学到非显然的事必须沉淀成 sub-skill 文件。**值得沉淀**：私有 API、稳定 selector / 命令、框架怪癖、URL 或路径模式、wait 的原因、陷阱。**不该写**：像素坐标、任务流水账、密钥/会话状态。

---

## 2. 必备结构

最小合格 skill 的目录树：

```
<name>-skill/
├── SKILL.md             ← 入口文档，必须提到三条原则与 self-heal 协议
├── helpers.*            ← agent 可读可改的源文件（任何语言）
├── interaction-skills/  ← 可初始为空，目录必须在
├── domain-skills/       ← 可初始为空，目录必须在
└── LICENSE
```

`SKILL.md` 必须包含：
- 一段话说明这个 skill 操作什么系统、用什么传输
- 一句"先读 helpers" 提示
- 一句"允许 self-heal" 授权
- 一句"必须 contribute back" 要求
- 链接回本 SPEC

---

## 3. Self-heal 协议

agent 在 helpers 不全时的 4 步标准动作：

1. **直接编辑** helpers 源文件，新增缺失函数
2. **优先复用最薄的传输层**——不要为补一个函数引入新框架或新依赖
3. **改完立即用**，不写测试套件、不开 PR review 流程
4. **任务结束前判断**该改动/发现是否值得沉淀回 sub-skill

---

## 4. Contribute-back 规则

每次任务结束前回顾：本次有没有学到非显然的东西？有就写 sub-skill。

写哪一层：
- 跨目标都成立的机制 → `interaction-skills/`
- 只对某个具体目标成立 → `domain-skills/<target>/`

写什么：私有 API 形状、稳定选择器/命令、框架怪癖、URL 模式、必要的 wait 及原因、陷阱与不该用的选择器。

不写：当次任务流水账、像素坐标、密钥、cookie、用户名、为"未来需求"的提前抽象。

---

## 5. 常见反模式

skill 容易被做"厚"，以下是高频违规：

1. 给 helpers 加 retry / circuit-breaker / config 系统
2. 把 helpers 变成只读 SDK，agent 不许改
3. 让人类手写 sub-skill 文件，而不是让 agent 沉淀
4. 把任务流水账（"今天我做了 ABC"）当 sub-skill 提交
5. 在 sub-skill 里写像素坐标、cookie、token、用户名
6. 为"未来可能的需求"提前抽象（违反 YAGNI）
7. 加 manager / supervisor / session-pool 层
8. 把 SPEC 内容复制进 skill（应当现拉，避免漂移）

---

## 6. License & contributing

MIT。欢迎 PR——优先：反模式补充、命名/结构歧义澄清。
