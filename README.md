# Owner-Review.skill

> **[English](#english)** | **[中文](#中文)**

---

<a id="english"></a>

**Let an AI agent review your changes like the repo owner would — catching blind spots and protecting existing functionality before code is written.**

> Stop worrying about breaking production. Start reviewing specs like someone who actually owns the codebase.

---

## The Problem

AI coding agents today have two dangerous tendencies:

1. **Tunnel vision.** Tell the agent to add something, and it only touches the exact spot you pointed at — ignoring every other part of the system affected by the change.
2. **Blind obedience.** Tell the agent to change something, and it changes it — even if the modification silently breaks three existing features.

These aren't bugs. They're fundamental design gaps. The agent has no sense of ownership over the codebase. It doesn't think about blast radius. It doesn't push back.

## The Solution

**Owner-Review.skill** gives your AI agent the perspective of a **repo owner** — someone who knows every flow, every dependency, every silent assumption in the codebase.

Built on the **SDD (Spec-Driven Development)** workflow:

1. You write a **spec or plan** describing the intended change.
2. Your AI CLI (e.g. Claude Code) loads this skill.
3. The agent reviews your spec **as if it were the repo owner** — tracing every affected feature flow, auditing shared modules for overlooked callers, and calling out every gap the spec missed.

The result: a structured **Verdict Report** that tells you exactly what your spec got right, what it missed, and what it'll break — **before a single line of code is written**.

## Who Is This For

- **New team members** who don't yet have full mental models of the codebase. No more accidentally breaking systems you didn't know existed.
- **Solo developers** who want a second pair of eyes with perfect memory.
- **Tech leads** who want spec reviews to catch real issues, not just formatting nits.
- **Any team practicing SDD** that wants automated, owner-grade spec review.

## How It Works

```
Phase 0: Confirm which spec(s) to review
    ↓
Phase 1: Inventory features the spec could affect
    ↓
Phase 2: Trace each feature's full logic flow via Search subagents
    ↓
Phase 3: Interrogate the spec against every traced flow
    ↓
Phase 4: Write a structured Verdict Report → saved to disk
```

The agent doesn't just list files — it thinks in **features and logic flows**. It independently reads the code to verify impact, never trusting the spec's own claims about what it affects.

## Quick Start

### Recommended: Claude Code (or any subagent-capable CLI)

For maximum effectiveness, use a harness CLI that supports **subagent dispatch** (e.g. [Claude Code](https://docs.anthropic.com/en/docs/claude-code)). The skill leverages Search subagents to trace feature flows in parallel with isolated context — this is where the real power comes from.

1. Clone or copy this skill into your project:

   ```bash
   # Copy the skill directory to ~/.claude/skills/
   mkdir -p ~/.claude/skills/
   cp -r skills/owner-review ~/.claude/skills/
   ```

2. Write your spec or plan document.

3. Invoke the skill and point it at your spec:

   ```
   > /owner-review review my spec at docs/specs/new-feature.md
   ```

4. The agent traces every affected feature flow, grills your spec, and saves a **Verdict Report** next to the spec file (e.g. `new-feature-review.md`).

### Other CLI Tools

Any AI coding CLI that supports loading custom skills/prompts can use this skill. You'll still get the owner-perspective review — subagent dispatch is optional but recommended for large codebases.

## What the Verdict Report Looks Like

```markdown
# Repo Owner's Verdict: [Spec Name]

## Summary
- Features: 8 analyzed — 2 high, 3 medium, 3 low
- Verdict: "The auth changes look solid, but the spec completely 
  ignores the session refresh flow. That WILL break."

## Feature Impact
### [FP-1] User Login — High
- Now: authenticates via email/password, returns JWT
- After spec: adds 2FA check before token issuance
- Flow diff:
  - Step 3: now also checks 2FA token ← NEW
  - Step 4: claims include `mfa_verified` ← CHANGED
- Ripple: FP-3 (session refresh), FP-7 (API key auth)

## What the Spec Missed
- **SessionRefreshHandler.refresh()** — still reads old JWT claims 
  without `mfa_verified`. Will throw on refreshed tokens. Severity: High.

## Risks
- Session refresh flow will break for all 2FA-enabled users
- API key auth bypasses 2FA entirely — spec doesn't address this
```

## Project Structure

```
skills/owner-review/
├── SKILL.md              # Main skill definition and workflow
└── analysis-prompt.md    # Search subagent prompt template
```

## Key Design Principles

- **Think in features, not files.** The review is organized by business capabilities, not by code symbols.
- **Never trust the spec.** The agent independently reads code to verify impact — the spec's own analysis is just a starting hint.
- **Trace full flows.** From trigger to response, including error paths and side effects. Specs always break edge cases silently.
- **Audit sibling methods.** When a flow touches a shared module, the agent reads the entire class — catching callers the spec author didn't know about.

## License

MIT

---

<a id="中文"></a>

# Owner-Review.skill

**让 AI Agent 以仓库 Owner 的视角审查你的改动 —— 在写代码之前，找出盲区、守护现有功能。**

> 新人再也不用担心把系统搞崩溃了。

---

## 痛点

当前的 AI 编程 Agent 有两个危险倾向：

1. **只看局部。** 你让它在某处加功能，它就只改那一个地方 —— 完全无视系统中其他受影响的部分。
2. **盲目服从。** 你让它改什么就改什么，即使这个改动会悄悄破坏三个已有功能，它也照做不误。

这不是 bug，而是根本性的设计缺陷。Agent 对代码库没有"主人翁意识"，不会考虑影响范围，也不会反驳你。

## 解决方案

**Owner-Review.skill** 赋予你的 AI Agent 一个 **仓库 Owner 的视角** —— 像一个了解每条流程、每个依赖、每个隐式假设的人一样审查改动。

基于 **SDD（Spec 驱动开发）** 工作流：

1. 你撰写一份 **spec 或 plan**，描述计划中的改动。
2. 你的 AI CLI（如 Claude Code）加载此 skill。
3. Agent **以 repo owner 的身份** 审查你的 spec —— 追踪每个受影响的功能流、审计共享模块中被遗漏的调用方，指出 spec 中每一处缺漏。

最终产出：一份结构化的 **审查报告（Verdict Report）**，在 **写下任何一行代码之前**，告诉你 spec 哪里写对了、哪里遗漏了、哪里会出问题。

## 适用人群

- **新人 / 新加入团队的成员**：还不熟悉代码库的全貌？不用怕了，再也不会误伤你不知道存在的系统。
- **独立开发者**：想要一个拥有完美记忆的"第二双眼睛"。
- **技术负责人**：希望 spec review 能抓到真问题，而不只是格式吹毛求疵。
- **任何实践 SDD 的团队**：希望拥有自动化的、owner 级别的 spec 审查。

## 工作流程

```
阶段 0：确认审查哪份 spec
    ↓
阶段 1：盘点 spec 可能影响的功能
    ↓
阶段 2：通过 Search 子 Agent 逐一追踪每个功能的完整逻辑流
    ↓
阶段 3：用每条已追踪的流程拷问 spec
    ↓
阶段 4：生成结构化审查报告 → 保存到磁盘
```

Agent 不是罗列文件 —— 它以 **功能和逻辑流** 为维度进行思考。它独立阅读代码来验证影响范围，绝不轻信 spec 自己声称的影响清单。

## 快速上手

### 推荐：Claude Code（或其他支持子 Agent 调用的 CLI）

为了发挥最大功效，建议使用支持 **子 Agent 调度** 的 CLI 工具（如 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)）。该 skill 利用 Search 子 Agent 在隔离上下文中追踪功能流 —— 这才是真正的威力所在。

1. 将此 skill 复制到你的项目中：

   ```bash
   mkdir -p ~/.claude/skills/
   cp -r skills/owner-review ~/.claude/skills/
   ```

2. 编写你的 spec 或 plan 文档。

3. 调用 skill 并指向你的 spec：

   ```
   > /owner-review 审查我的 spec：docs/specs/new-feature.md
   ```

4. Agent 会追踪所有受影响的功能流，拷问你的 spec，并将 **审查报告** 保存在 spec 旁边（如 `new-feature-review.md`）。

## 核心设计原则

- **以功能为维度，而非文件。** 审查按业务能力组织，而不是按代码符号。
- **绝不信任 spec。** Agent 独立阅读代码验证影响 —— spec 自己的分析只是一个起点。
- **追踪完整流程。** 从触发到响应，包括错误路径和副作用。Spec 永远会悄悄打破边界情况。
- **审计兄弟方法。** 当流程经过共享模块时，Agent 会阅读整个类 —— 找出 spec 作者不知道的调用方。

## 开源协议

MIT
