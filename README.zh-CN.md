# output-contract

[English](README.md) | 简体中文

**一个让 AI 编程助手不再到处乱丢文件的 Agent Skill。**

用过 Claude Code、Codex CLI、ZCode、Cursor 这类编程助手的人都见过这个场面：这里一个临时分析脚本、那里一个 `run.log` 和一张 `fig1_final.png`，仓库根目录再躺三个 `test_tmp.py`——因为 agent 写文件时永远落在"当前工作目录"里。事后清理的成本，远高于开工前花三十秒约定好放哪。

这个 skill 就是让 agent 做到这一点：

1. **开工前** —— 用一条消息声明契约：唯一维护位置，以及代码／日志／交付物／暂存文件各放哪。已有项目的目录结构优先于默认模板；全新的独立工作默认用 `{主题}_{日期}/`，下设 `scripts/`、`logs/`、`outputs/`、`scratch/`。
2. **执行中** —— 输出路径集中定义、从声明的根目录派生、运行时解析为绝对路径，落点不随启动目录漂移。版本纪律：代码原位维护（历史交给 git），不堆 `01_…/02_…/final_2.py`；需要留存的运行结果进 `runs/<运行标识>/`。
3. **收尾时** —— 用产物清单（`_index.md`）核对实际写入路径，报告目录外文件和重复副本；暂存成果转正进项目时走带验证的交接流程——"已复制"不等于"已整理完成"。

完整契约见 [`SKILL.zh-CN.md`](SKILL.zh-CN.md)（中文）/ [`SKILL.md`](SKILL.md) (English)。

## 安装

任何按 `SKILL.md` 规范加载 skill 的 agent（Claude Code、Codex CLI、ZCode 等）都能用。克隆仓库后作为一个 skill 暴露出来：

**macOS / Linux**

```bash
git clone https://github.com/bee014108/output-contract.git
mkdir -p ~/.agents/skills
ln -s "$(pwd)/output-contract" ~/.agents/skills/output-contract
```

**Windows（Git Bash / PowerShell）**

```bash
git clone https://github.com/bee014108/output-contract.git
mkdir -p ~/.agents/skills
cp -r output-contract ~/.agents/skills/output-contract
```

（也可以拷进各家工具自己的 skill 目录：Claude Code 用 `~/.claude/skills/`，Codex CLI 用 `~/.codex/skills/`。）

想用中文版契约？安装后把 `SKILL.zh-CN.md` 改名为 `SKILL.md`（原英文版移走）即可。

## 它和更强制的手段是什么关系

本 skill 属于**约定层**：靠指令改变 agent 的行为。在此之上还有两层更强的强制，可以自由叠加：

- **Hook 层** —— 例如 Claude Code 的 `PreToolUse` hook 可以硬拦截写到白名单目录之外的 `Write`/`Edit`。
- **操作系统沙箱层** —— 例如 Codex CLI 的 `sandbox_mode = "workspace-write"` 加 `writable_roots = [...]`，在系统层面限制可写范围。

这两层管住的是"**不许写错地方**"；本 skill 还补上了它们不管的那半边：**产物清单、版本纪律、收尾核对**——让写出来的东西事后找得到、整理得动。

## 许可证

[MIT](LICENSE)
