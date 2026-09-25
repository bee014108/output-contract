# output-contract

**An Agent Skill that stops AI coding agents from scattering files all over your workspace.**

If you use Claude Code, Codex CLI, ZCode, Cursor, or any coding agent, you know the pattern: a quick analysis script here, a `run.log` and a `fig1_final.png` there, three `test_tmp.py` in the repo root — because agents write wherever the current working directory happens to be. Cleaning up afterwards costs far more than agreeing on destinations *before* the first line of code.

This skill makes the agent do exactly that:

1. **Before work starts** — declare a contract in one message: the single maintenance location, and where code / logs / deliverables / staging files will go. Existing project layouts win over defaults; genuinely new standalone work gets `{topic}_{date}/` with `scripts/`, `logs/`, `outputs/`, `scratch/`.
2. **During execution** — output paths are defined centrally, derived from the declared root, and resolved to absolute paths, so destinations never drift with the launch directory. Version discipline: code is maintained in place (history belongs to git), not by accumulating `01_…/02_…/final_2.py`; run results worth keeping go to `runs/<run-id>/`.
3. **At wrap-up** — reconcile the manifest (`_index.md`) against actual writes, report out-of-tree files and duplicate copies, and complete a verification-backed handover when promoting staged work into a project. "Copied" is not "organized".

The full contract is in [`SKILL.md`](SKILL.md) (English) / [`SKILL.zh-CN.md`](SKILL.zh-CN.md)（中文版）.

## Install

Any agent that loads `SKILL.md`-style skills (Claude Code, Codex CLI, ZCode, …) can use it. Clone the repo and expose it as a skill:

**macOS / Linux**

```bash
git clone https://github.com/bee014108/output-contract.git
mkdir -p ~/.agents/skills
ln -s "$(pwd)/output-contract" ~/.agents/skills/output-contract
```

**Windows (Git Bash / PowerShell)**

```bash
git clone https://github.com/bee014108/output-contract.git
mkdir -p ~/.agents/skills
cp -r output-contract ~/.agents/skills/output-contract
```

(Or copy into your tool's own skills directory: `~/.claude/skills/` for Claude Code, `~/.codex/skills/` for Codex CLI.)

Prefer the Chinese version? Rename `SKILL.zh-CN.md` to `SKILL.md` (swap the English one out) after copying.

## How it relates to harder enforcement

This skill is the **convention layer**: it changes agent behavior by instruction. Two stronger layers exist and combine freely with it:

- **Hooks** — e.g. Claude Code `PreToolUse` hooks can hard-block `Write`/`Edit` outside approved directories.
- **OS-level sandboxing** — e.g. Codex CLI's `sandbox_mode = "workspace-write"` plus `writable_roots = [...]` limits writes at the OS level.

Those layers stop *wrong* writes; this skill also handles the part they don't: the **manifest, version discipline, and wrap-up reconciliation** that make artifacts findable and reorganizable afterwards.

## License

[MIT](LICENSE)
