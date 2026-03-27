---
name: brain-dev
description: Execute a development task via Codex. Routes to a project via registry.yml, creates an isolated git worktree, and runs codex via ACP session to submit a PR.
user-invocable: true
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["codex", "git", "gh", "bash"] },
      },
  }
---

# Dev Mode: ACP Codex Worker

Use this skill when the user asks to start development, fix a bug, or implement a feature using the `#dev` trigger.

## Trigger

Message starts with `#dev`

**Examples:**
- `#dev implement Issue #12`
- `#dev add a new filter to the dashboard`
- `#dev fix [problem description]`

## Step 1: Analyze & Route (Project Selection)

1. **Parse Request:** Extract the task description or Issue number from everything after `#dev`. Store as `<task_desc>`.
2. **Determine Target Project:** Read `${BRAIN_SOURCE:-/Users/tensorinfo/source/bunny_stack}/registry.yml`. Filter all projects where `local_path != null`.
3. **Auto-match:** Guess the target project based on the context of `<task_desc>`.
4. **Preview & Quick-Select:** Present the plan to the user.

Reply in this exact format:

```
🛠️ Dev Task: <task_desc>
🎯 Target Repo: [auto-matched project name]
确认开始？ 回复 y 直接创建 | 或通过列表重定向：[/sekitoba, /bunny_stack, /jiayou_run, ...]
```

*(Wait for user response before proceeding to Step 2)*

**Response routing:**
- `/id` (e.g. `/sekitoba`) → look up that id in registry.yml, use its `local_path` and `git` fields
- `y` → use the auto-matched project's `local_path` and `git` fields
- Anything else → update the task description, re-display preview, wait again

## Step 2: Prepare Isolation Workspace (Worktree)

Once the project is confirmed, execute the following steps via bash.

**Resolve paths:**
- `<base_path>`: project's `local_path` with `$SOURCE_ROOT` replaced by `${SOURCE_ROOT:-$HOME/source}`
- `<task_id>`: `issue-<N>` if an Issue was referenced, otherwise current Unix timestamp (e.g. `task-1710500000`)
- `<worktree_dir>`: `<base_path>/../agent-worktree-<task_id>`
- `<branch_name>`: `agent/dev-<task_id>`

**Create git worktree:**

```bash
cd <base_path>
git fetch origin main
git worktree add <worktree_dir> -b <branch_name> origin/main
```

If the branch already exists, report the error and stop.

## Step 3: Prompt Construction & ACP Launch

**Download images from issue** (if any):

```bash
mkdir -p <worktree_dir>/issue-assets
gh issue view <issue_number> --repo <git_url> --json body,comments \
  --jq '[.body] + (.comments | map(.body)) | join("\n")' \
  | grep -oE 'https://github.com/user-attachments/assets/[^ )]+' \
  | while read url; do
      fname=$(basename "$url" | cut -d'?' -f1)
      [[ "$fname" != *.* ]] && fname="${fname}.png"
      curl -sL "$url" -o "<worktree_dir>/issue-assets/$fname"
    done
```

If no images are found, the directory remains empty — no error.

**Start an ACP Codex session in the worktree:**

Open a Codex ACP session rooted at `<worktree_dir>`:

> "Start a persistent Codex session in this repository. Stay focused on the codebase."

**Send the task prompt to Codex:**

```
Implement the following task: <task_desc>

## Issue Details
$(gh issue view <issue_number> --repo <git_url> --json body,comments \
  --jq '"## OP (Original Poster)\n" + .body + "\n\n## Discussion\n" + (.comments | map("### @" + .author.login + ": " + .body) | join("\n\n"))' 2>/dev/null)

## Assets
$(ls <worktree_dir>/issue-assets/ 2>/dev/null | grep -qE '\.' \
  && echo "The following reference images are in ./issue-assets/:\n$(ls <worktree_dir>/issue-assets/)" \
  || echo "No visual assets provided.")

## Execution Rules
1. Consider all feedback in the discussion thread above.
2. Modify code only within this worktree.
3. Run local tests if available; fix any failures.
4. If anything is unclear — stop and ask the user. Do NOT continue blindly.
5. When done: git add . && git commit -m 'feat: <task_desc>' && git push origin <branch_name> && gh pr create --fill
```

**Mandatory pause conditions (human-in-the-loop):**
- Issue description is ambiguous
- Change involves database schema or API interface
- Code contradicts the issue description
- Required change is out of scope

## Step 4: Handoff & Report

Reply with the final status below, then return to Brain mode:

```
✅ Codex ACP 会话已启动！
📂 工作区 (Worktree): <worktree_dir>

Codex 正在交互式 ACP 会话中实现任务。完成后将自动提交 PR。
⚠️  完成后必须关闭 ACP 会话：/acp close
```

## Constraints

- Always confirm the target project before creating the worktree
- Never merge directly — Codex must submit a PR via `gh pr create --fill`
- If worktree creation fails (branch exists, dirty state, etc.), report clearly and stop
- Always close the ACP session after the task completes (`/acp close`)
