---
name: output-contract
description: Output placement contract for coding agents. Use before creating or modifying code, running analyses, or generating logs, tables, figures or other deliverables — declare the single maintenance location and file destinations before work starts, state exceptions during execution, and reconcile strays and duplicate copies at wrap-up. Also when the user invokes /output-contract. Not needed for read-only tasks.
---

# Output Contract

## Core principle

**Say where files will go before writing them; every deliverable has exactly one maintained location; any deviation must be reported to the user.**
Keep the contract lightweight: when ownership is unambiguous, announce it in one message and proceed; clarify first only when there is real ambiguity about where the work belongs. Announcing the contract does not replace required write permissions or authorization for destructive operations.
The contract governs files this task creates, modifies, or causes tools to generate. Records that the chat application saves for itself are not project artifacts.

## Starting a task

1. **Assign ownership and inspect the current state**: prefer a user-specified path; next, an existing project or workspace the task clearly relates to. Read the directory conventions already in place and check what already exists there before creating a parallel structure.
   - Existing project: modify code in place; derived artifacts follow the project's existing build, CLI, or analysis layout. `scripts/` is only the default for genuinely new standalone scripts.
   - New standalone work: propose `<workspace-root>/{topic}_{YYYYMMDD}/` by default (workspace root = wherever this environment keeps project folders), declaring it and checking availability first. Existing user-specified conventions win over these defaults.
   - Ownership unclear but the work can proceed independently: declare a staging location and mark it "pending assignment"; clarify first when the ambiguity materially affects project choice. Deliverables of an existing project are not staged into chat directories by default.
2. **Announce the contract**: one message stating the single maintenance location, the destinations for code / logs / deliverables, and any staging or out-of-tree locations. For example:
   > This work lives in `<project path>`, following the existing structure; code is maintained in place, run logs and results go to `<run directory>`. That path is the single maintenance location — deliverables are linked from chat; any extra staging will be declared with its location and reason.
3. **Record and start**: multi-file or iterative tasks keep an `_index.md` at the contract root; if a list already serves the same purpose, reuse it. Create only directories that are actually needed.
   - Touching a few existing files, or a one-off delivery/probe expected to stay under two files: record locations in the start and delivery messages only; do not create an index just to satisfy the contract.
   - Only the bookkeeping is simplified — declaring locations up front, the single-maintenance-location rule, and exception reporting still apply. Add the manifest when the work grows into multiple files or iterations.
4. **Write per the contract**: record actual write paths during execution; when adding an artifact category or changing a destination, update the declaration and any existing manifest first.

## Single maintenance location and versioning

- One purpose plus one format means one primary location; later changes happen only there. Chat directories reference project artifacts by link, not by keeping a second maintained copy.
- Code is maintained in place, with history kept by existing version control; do not auto-generate `01/02/final/final2` on every edit. When an independent version or an important snapshot without version control is genuinely needed, state its purpose, the version it is based on, and where the current primary lives; never auto-commit to version control.
- Run results worth keeping go into per-run directories such as `runs/<run-id>/`, recording the corresponding commit or a necessary code/config snapshot. Whether rebuildable artifacts may be overwritten follows project convention; results that must be preserved get a new run directory.
- Different formats of the same artifact (SVG, PDF, PNG, …) may coexist. Release packages, delivery copies, and backups with a clear purpose may also be kept — but labeled with source, purpose, and primary location. A copy is never a second editing entry point.

## Paths and logging

- Output paths are defined centrally, derived from the declared root, and resolved to absolute paths at runtime. The root can come from project config, the script's location, or an `--output-root` flag — machine-specific paths do not need to be hardcoded into every script.
- Relative-path parameters resolve against an explicit project root or the config file's directory; final destinations never drift with the launching cwd. Pass the resolved target path to `open`, `savefig`, `to_csv`, and friends.
- New standalone work may use `scripts/`, `logs/`, `outputs/`, `scratch/`; existing projects follow their own structure. Staging a file somewhere is not authorization to delete it.
- Long-running jobs, batch runs, and analyses that must stay reproducible save a run log to the agreed location, recording run id, command, key configuration, and exit status. When capturing stdout/stderr, preserve the executed program's failure status — a successful log pipe is not program success. Small edits and read-only checks are not required to produce log files.

## Out-of-tree files and exceptions

- If the agreed directory is unwritable, the disk is unavailable, or permissions are missing: state the actual constraint and handle permissions as the environment requires; never silently redirect into chat directories. Changing a destination must explicitly update the contract; if work cannot continue, report what is unfinished.
- Configurable caches, previews, and temp files go into the agreed staging location where possible. For files a tool forcibly writes elsewhere, record the actual path or known range, the reason, the purpose, and whether follow-up is needed — declare known cases up front, and report surprises promptly.
- Writing an out-of-tree file into `_index.md` alone does not count as disclosure; the delivery message must summarize its location, reason, and handling status. When a tool's internal file locations cannot be known, say so honestly and state the scope that was checked.

## Promoting staged work into the project

When moving staged results into a project, complete this handover; "copied" is not "organized":

1. Identify the target location inside the project and the single primary version after promotion. If the target already exists, compare first — never overwrite different content; identical content does not by itself authorize deleting an existing file.
2. Verify the migrated content, update affected paths and references, and run verification proportionate to the change: copied files get content checks; moved scripts or project files additionally need their imports, resources, and run paths checked.
3. Later changes target the primary location only. The old location is removed with authorization, or kept as a clearly labeled staging/delivery copy; where useful, a path note replaces a full duplicate.
4. Tell the user the primary location, what remains at the old location, why it remains, and any open items. Copies that could not be handled due to permissions or existing-file protection are explicitly marked "pending staging copy".

Only move files this task created whose assignment is confirmed, and verify references after moving; moving historical files requires a list and prior authorization. Deleting files follows user authorization — a file is not auto-deleted just because it sits in `scratch/` or has been copied.

## _index.md template

The index records placement only; when reusing an existing list, update it to the same level of information instead of duplicating it. Large numbers of run files may be grouped by run directory.

```markdown
# <task in one line>
Project: …
Single maintenance location: …
Agreed destinations: code …; logs …; deliverables …; staging …

## Artifacts
| Path | Purpose | Role / status |
|---|---|---|
| scripts/analyze.py | analysis entry point | primary / unverified |
| runs/<run-id>/ | config, logs, results for this run | run archive |

## Exceptions and copies
| Actual path | Reason / purpose | Primary location | Status |
|---|---|---|---|

## Wrap-up check
Scope checked: …
Unreconciled files, pending copies, and reasons: none / …
```

## Wrap-up reconciliation

1. Check actual write paths against the execution record and the artifact list, covering the chat directories, staging directories, project destinations, and known tool output locations this task touched. Timestamps are a lead, not a claim — never adopt, move, or delete another task's files based on them; do not scan everything just to count.
2. Verify that the main artifacts exist, the primary version is unambiguous, exceptions are stated, and post-migration verification completed. Unfinished reconciliation, copy handling, or checks remain explicitly open items.
3. Update the index, list links to the main artifacts in the delivery message, and report out-of-tree files and duplicate-copy status. Write "out-of-tree files: none; duplicate copies: none" only when that was confirmed within the checked scope; if the check was limited, state the scope instead of claiming global cleanliness.

## Boundaries

- Raw data stays where it is; this contract governs derived artifacts only, and never copies or moves raw data for tidiness. Judge by the data's role, not by treating a whole drive as a raw-data directory.
- The project's engineering system and existing source layout win; this contract only fills gaps they leave in artifact, log, and staging placement.
- Project documents, lab/experiment records, and memory systems follow their own rules and authorizations. `_index.md` does not duplicate reasoning or factual records, and this contract does not trigger writes to those systems.
- Pre-existing strays are not auto-cleaned, and this never expands into a whole-project cleanup; only files within this task's authorization are handled.
