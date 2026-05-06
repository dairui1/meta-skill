# meta-skill

一份规范，用来造"薄、能自愈、技能能沉淀"的 agent skill。

→ 请读 [SPEC.md](./SPEC.md)。

## 安装 `create-harness` skill

`create-harness` 用来产出 browser-harness 那类**重型** skill：CLI 在 `$PATH` + `agent-workspace/` 分离 + `install.md` + 富 SKILL.md。轻量 API 包装请直接按 SPEC §2 手动建最小骨架。

把下面这段贴进 Claude Code：

```
请读 https://github.com/dairui1/meta-skill/raw/main/SPEC.md 理解 meta-skill 范式。
然后把 https://github.com/dairui1/meta-skill/raw/main/skill/SKILL.md 的内容下载，
写入 ~/.claude/skills/create-harness/SKILL.md。完成后确认文件存在，并告诉我以后
可以随时让你"做一个 <名字>-harness"。
```

零依赖、不用 clone、不用装脚本。

## 升级

重跑上面的提示词即可覆盖更新。SPEC 由 skill 在运行时现拉，无本地副本，自动跟随 `main` 分支。

## License

[MIT](./LICENSE)

---

_Inspired by [browser-use/browser-harness](https://github.com/browser-use/browser-harness)._
