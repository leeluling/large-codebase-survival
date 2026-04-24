# 中文社区帖子草稿

## 发布平台：V2EX (程序员节点)、掘金、知乎

---

## 标题：
用 AI 编程助手改造 15000 行 Python 项目，崩溃了 20 多次后总结出的生存指南

## 正文：

### 背景

我有一个生产环境的桌面应用（I2D 客户端），核心代码是一个 15000+ 行的 Python 文件，包含 UI、业务逻辑、设备通信等所有功能。我需要把 UI 从 SimpleGUI 迁移到 WebView。

自然想到用 Claude Code 来辅助完成这个工作。但现实很残酷：

- 每次 session 都会因为 context overflow 崩溃（503 错误）
- 一旦 503，整个 session 永久死亡，连 `/clear` 都救不回来
- 之前做的所有进度全部丢失
- 反复了 20 多次，零进展

### 转折

经过反复摸索，我找到了一套方法论，让项目从"不可能完成"变成了"虽慢但稳步推进"。

### 核心思路

**把 AI agent 当项目经理用，而不是当开发者用。**

主 agent 永远不要直接读大文件。它的工作是：拆分任务 → 派发给 subagent → subagent 完成后保存 → 下一个任务。

如果 subagent 崩了，主 agent 还活着，可以派新的。进度已经保存了，新 session 也能接着来。

### 六条原则

1. **保护主 Agent** — 所有重活交给 subagent，主 agent 只做调度
2. **随时保存进度** — 每完成一个函数/模块就 commit，假设 session 随时会死
3. **最小化 context 消耗** — 只读需要的内容，控制输出长度
4. **大任务拆成小块** — 拆到每块能在一次 subagent 调用内完成
5. **预留 context 空间** — 别把上下文用满，留出余量
6. **为 session 中断设计** — 维护进度文件，新 session 能无缝接续

### 开源

我把这套方法论整理成了一个 GitHub 项目，包含：
- 完整的方法论文档（英文）
- 可以直接复制到 CLAUDE.md 的模板
- 进度跟踪模板

**GitHub:** https://github.com/leeluling/large-codebase-survival

MIT 协议，适用于 Claude Code、Cursor、Windsurf 等所有有 context 限制的 AI 编程工具。

欢迎 star、提 issue、分享你的经验。
