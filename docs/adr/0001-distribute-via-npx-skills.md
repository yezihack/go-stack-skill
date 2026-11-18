# Distribute skills through the npx skills installer

**Status:** accepted

The skill bundle is distributed by pointing users at this repository's README, which routes them to `npx skills add yezihack/go-stack-skill`. The repository keeps a single canonical copy under `skills/<name>/SKILL.md`; `.agents/skills/` is only an install destination. We deliberately do not ship per-platform manifests (Claude marketplace, Cursor rules, Gemini extensions) or a custom install script.

**Why:** the installer discovers this layout and supports 79 agents, including CodeBuddy, OpenClaw, and Hermes. One path keeps install, update (`npx skills update`), and documentation in sync, while per-platform manifests would add multiple sources of truth and drift.

**Considered options:** a Claude Code marketplace as the primary channel (best update story, but one platform only); a pure manual copy guide (no dependency, but fragile and verbose); a bundled install script (works offline, but OS-specific and requires trusting a downloaded script). All remain possible later without changing the canonical `skills/` layout.
