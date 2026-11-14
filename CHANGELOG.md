# Changelog

## 1.1.1 — 2026-11-15

修复 macOS 上「会话证据全为 0」的问题：归档会正常建目录，但归属、时间窗、写入证据都读不到，等于退化成瞎猜。

### 修复
- `session-inventory.mjs`：选转录文件时排除 `session.lock`。DSH 现在在会话目录里同时写 `session.lock` 与 `session.v4.jsonl.zstd`，`startsWith('session.')` 会命中字典序更靠前的空锁文件，解码得到空内容 → 所有会话 `created`/`last`/`writes` 归零。`auto-plan.mjs` 复用同一个 `scanStore`，`--auto` 一并修好。
- `session-inventory.mjs`：写入/提及证据改为跨平台提取。原正则只认 Windows 盘符（`C:\…`），macOS/Linux 下 `writes` 恒为 0；现在以会话自身 `cwd` 为起点截取 POSIX 路径，Windows 盘符规则保留。

### 说明
- 已在 macOS 实测：同一工作区 6 个会话的时间窗、首条用户消息与提及路径全部正确还原，`workspace-purge.mjs` 的 SHA-256 覆盖率校验正常出结论。

## 1.1.0 — 2026-11-05

新增自动归档模式：新会话里一条命令即可按会话时间窗分类落盘，不再需要手写 plan.json。

### 新增
- `scripts/auto-plan.mjs`：从会话库与工作区推断归档计划（会话选择、时间窗过滤、8 段式分类、保留原位判定）。
- `workspace-archive.mjs --auto`：一条命令完成「选会话 → 分类 → 复制 → 生成 README/总览」，支持 `--session` / `--all` / `--topic` / `--save-plan`。
- `references/归属与分类规范.md` 增加自动分类判定表（12 级优先级）与跳过规则。

### 安全
- 自动模式只复制不删除；清理仍需 `workspace-purge.mjs` 的 dry-run + SHA-256 校验。
- 跳过归档目录自身、含 `README_总览.md` 的目录、`skills`、`node_modules`/`venv`/`.git` 等噪声目录。

## 1.0.0 — 2026-11-05

首个可用版本，提炼自一次真实工作区整理（33 个会话 / 647 个归档文件 / 564 个原件清理）。

### 新增
- `SKILL.md`：7 步工作流（盘点 → 归属 → 计划 → 归档 → 校验 → 清理 → 收尾）与 7 条实战陷阱。
- `scripts/session-inventory.mjs`：按 zstd frame 边界解码多帧会话转录，输出会话清单、时间窗与写入证据。
- `scripts/workspace-archive.mjs`：按 plan.json 做 8 段式分类复制，并生成每个会话的 README 与总览。
- `scripts/workspace-purge.mjs`：SHA-256 覆盖率校验后删除原件，默认 dry-run。
- `references/归属与分类规范.md`、`references/清理检查清单.md`。
- `templates/plan.example.json`。

### 说明
- 三个脚本均支持 `--help`，仅依赖 Node 内置模块，要求 Node ≥ 22。
