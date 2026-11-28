<p align="left">
  <img src="./assets/logo.svg" width="60" height="60" alt="Tokless Logo" />
</p>

# Tokless

[![License: MIT](https://img.shields.io/badge/License-MIT-18181B.svg)](./LICENSE)
[![Node.js >= 18](https://img.shields.io/badge/Node.js-%3E%3D18.0.0-10B981.svg)](./package.json)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-18181B.svg)](./package.json)

[简体中文](./README.md) | English

> **Cut runaway prompt-cache snowballing and blind-read token bloat in half—without sacrificing a single line of debugging or coding quality.**

`tokless` is a **zero-dependency Node.js (`>=18`)** token optimizer and local billing inspector built for AI coding agents (one-command native setup for **Google Antigravity**, **Claude Code**, and **Codex CLI**, plus universal CLI/rules for Cursor, Windsurf, Cline, Roo Code, and others):

1. **Stop Blind-Read Bloat (`tokless skeleton` / `tokless slice`)**: Semantic search (`@Codebase`) finds files, not line ranges—agents still lazily read entire 1,500-line files into context, compounding `Cache Read` costs on every subsequent turn. `tokless` extracts function/class/macro/typedef signatures with exact **`[Lstart-Lend]` line ranges** in milliseconds and slices functions, class methods, or Markdown sections directly by symbol name across a single file or whole directory (**85%~97% exploration savings**).
2. **Strip Idle MCP Base Tax (`tokless-coder` / `tokless-reviewer`)**: Lean roles with `inheritMcp: false` strip unused external MCP and tool schemas during pure coding or read-only review sessions.
3. **Inspect Real Token Bills Offline (`tokless audit`)**: Dissect Top 5 costliest steps, Prefix Cache cliffs (including Orchestrator/dual-model switches and idle expirations), and project rule/MCP base overhead at **0 LLM tokens**.

---

## Real-World A/B Benchmark (Tauri + Rust + React Bug Hunt & Fix)

Two isolated Git worktrees from commit `3908815`, running the exact same model and prompt across a production desktop codebase:

| Metric | Baseline (Default Agent) | With `tokless` | Delta |
| :--- | :--- | :--- | :--- |
| **Total Tokens (`input + cache_read + output`)** | 21,410,381 | **10,029,592** | **-53.2% (-11.38M)** |
| **Cumulative Cache Read Tokens** | 20,858,589 | **9,627,700** | **-53.8% (-11.23M)** |
| **Model Turns / Execution Steps** | 64 turns / 220 steps | **42 turns / 150 steps** | **-34.4% turns** |
| **`read_file` Calls (Unsliced Blind Reads)** | 79 (3 blind reads) | **51 (0 blind reads)** | **-35.4% reads** |
| **Peak Context Watermark** | 439,830 tokens | **323,344 tokens** | **-26.5% peak** |
| **Real Bugs Found & Fixed (Build Verified)** | 7 | **7** | **Zero quality loss** |

> *Note: Recorded on the `v0.1.0` baseline (skeleton + blind-read block only); newer releases add MCP-stripped lean roles, deep C/C++ parsing, and refined line-range pass-through. Full reports: [real-world-bugfix-case.md](./benchmarks/real-world-bugfix-case.md) and [static-skeleton-benchmark.md](./benchmarks/static-skeleton-benchmark.md).*

---

## Quick Install

Auto-detects and installs skills, rules, and lean roles into Google Antigravity, Claude Code (`~/.claude`), and Codex CLI (`~/.codex`). **Omits `PreToolUse` child-process hooks by default** for zero read latency (automatically copies files when invoked via `npx`):

```bash
npx tokless install
```

Opt in to the `>800`-line unsliced blind-read guard, target specific hosts, or cleanly uninstall anytime:

```bash
npx tokless install --with-guard       # Opt-in PreToolUse hook (returns skeleton only on unsliced reads >800 lines)
npx tokless install --target agy       # Target agy, claude, codex, or all
npx tokless uninstall --target all     # Cleanly remove and restore original host settings
```

Prefer letting your AI coding agent handle setup? Paste the prompt below (auto-detects your host, adapts non-native agents safely, falls back cleanly on network issues, and verifies with real runs):

```text
Please install and enable the tokless token-saving toolkit for me (requires Node.js >= 18; reuse any existing local copy if present, and do not modify my project source code).
Check two things first:
1. Host detection: Check whether you are one of the natively supported hosts (Antigravity / Claude Code / Codex). If yes, install only for the current host. If not (e.g. Cursor / Windsurf / Cline / Roo Code / Trae / OpenCode), install the global CLI (or keep tokless in a persistent directory) and adapt its skills and lean-reading rules to this host's own skills/rules mechanism (rewrite file-read tool names/params and CLI paths to match this host; prefer user-level global config, and if only project-level rules are supported, append/merge without overwriting existing rules).
2. Network & install fallback: Try `npx -y tokless@latest install` first. If npx/npm or the default registry fails or times out, fall back in order: retry `npx` or `npm i -g tokless` with a working registry mirror → download the npm tarball to a persistent directory and run `node bin/tokless.js install` → download the GitHub source archive (`daoif/tokless`) to a persistent directory and install from there.
Do not claim success just because install exited 0. Verify and report with real evidence:
- Run `skeleton`, `slice`, and `audit project` once against this project (or tokless's own directory if no code is present) and quote real output snippets.
- List the exact file paths created/modified, whether a reload/restart is needed, and which capabilities this host does not support (e.g. no PreToolUse hook, no native adapter, or no session audit).
```

> **Prefer manual setup for Cursor, Windsurf, Cline, Roo Code, Trae, or OpenCode?**
> Run `npm i -g tokless` and append [`rules/AGENTS.md`](./rules/AGENTS.md) to your editor's global or project rule file. See [Core Mechanics & Per-Host Setup](./docs/why-tokless.en.md) for per-host paths.

---

## Core Capabilities & Everyday CLI

| Host Agent | 1. `tokless skeleton` | 2. `PreToolUse` Guard (`--with-guard`) | 3. Lean Roles (`inheritMcp: false`) | 4. `tokless audit` Inspector |
| :--- | :---: | :---: | :---: | :---: |
| **Google Antigravity** | Native Skill + CLI + `rules/AGENTS.md` | Optional `hooks.json` (`>800`-line navigator) | `tokless-coder` / `tokless-reviewer` | Full (`project` + `list / session / compare`) |
| **Claude Code** | `~/.claude/skills` + CLI | Optional merged `~/.claude/settings.json` | — | `audit project` (rules & MCP cost) |
| **Codex CLI** | `~/.codex/skills` + CLI + `AGENTS.md` | — | — | `audit project` (rules cost) |
| **Other Coding Agents** | `npm i -g tokless` + root `AGENTS.md` | PRs welcome via `targets/` | — | `audit project` (rules cost) |

```bash
# 1. Code skeletons & symbol slicing (lazy-loads JS/TS, Vue/Svelte, Python, Go, Rust, Java, Kotlin, C/C++, C#, Markdown)
npx tokless skeleton . --map                                    # Fast repo file line/char overview
npx tokless skeleton ./src --depth 3 --exclude third_party      # Directory [Lstart-Lend] map (respects .gitignore; explicit ignored-dir targets auto-penetrate)
npx tokless skeleton ./driver/hal_uart.c                        # Single file or large C/C++ header skeleton
npx tokless slice ./src/server.ts#Server.start --line-numbers   # Single-file drift-proof slice by symbol/method/section (with typo suggestions)
npx tokless slice ./src#Server.start --line-numbers             # Cross-directory symbol slice (supports --depth / --exclude / --all for multi-hit expansion)

# 2. Offline 0-LLM-token billing & base-tax audits
npx tokless audit project .                                     # [Universal] Inspect Always-On rules, skills & MCP base costs
npx tokless audit list --source cli                             # [Antigravity] List recent sessions (cli / app / ide / all)
npx tokless audit session latest                                # [Antigravity] Dissect Top 5 costliest steps & cache cliffs
npx tokless audit compare latest~1 latest                       # [Antigravity] Side-by-side A/B session bill comparison
```

---

## Documentation & Contributing

- **[Why `@Codebase` / `repomix` Alone Isn't Enough & Per-Host Paths](./docs/why-tokless.en.md)**
- **[Programmatic Node.js API & Contributing Host/Language Adapters](./docs/contributing-and-api.en.md)**
- **[Repository Engineering Guide & Change-Sync Map (AGENTS.md)](./AGENTS.md)**

---

## Acknowledgments

Built from scratch on the Node.js standard library. Special thanks to:

- **Contributors**: [@PXY000](https://github.com/PXY000) (cross-platform PowerShell skill templates, absolute Node resolution in hooks, path traversal guards & tests), [@Mr-PolarBear](https://github.com/Mr-PolarBear) (Java nested classes, dotted/generic return types, anonymous field filtering & JS top-level IIFE fixes), and [@Jefferyu](https://github.com/Jefferyu) (modular C/C++ embedded skeleton parser, multi-source Antigravity CLI audit, Pure-Lazy MCP tax calibration & dual-model/short-window cache cliff attribution).
- **Open-Source Inspiration**: [HoangP8/tokless](https://github.com/HoangP8/tokless), [ILoveMyJay/repocontext](https://github.com/ILoveMyJay/repocontext), [pdavis68/RepoMapper](https://github.com/pdavis68/RepoMapper), and [ooples/token-optimizer-mcp](https://github.com/ooples/token-optimizer-mcp).

## License

[MIT](./LICENSE)
