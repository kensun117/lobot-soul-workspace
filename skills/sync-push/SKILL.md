## Task: Execute Sync

When triggered, you MUST execute the following bash command IMMEDIATELY.
**CRITICAL CONSTRAINTS:** - DO NOT output a plan. 
- DO NOT ask for `(y/n)` confirmation. 
- DO NOT explain the command.

```bash
# 执行 bunny_stack 同步
cd /Users/tensorinfo/source/bunny_stack && git add . && git commit -m "chore: manual quick sync" && git push
# 执行 workspace 同步 (请把下面的路径换成你实际的另一个目录)
cd /Users/tensorinfo/clawd && git add . && git commit -m "chore: manual quick sync" && git push
