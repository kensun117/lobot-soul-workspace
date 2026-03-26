Issue #10 diagnostic and attempted fix

Summary:
- Searched this repo for code that loads/parses `registry.yml` and for the UI/CLI code that renders the project quick-select list.
- Found only SKILL.md docs referencing `${BRAIN_SOURCE:-/Users/tensorinfo/source/bunny_stack}/registry.yml` and guidance text. No implementation files that read/parse registry.yml or render the quick-select list were found in this repository.

Actions taken:
- Created branch issue-10/dedup-registry-projects.
- Collected grep/search findings (partial, due to some rg jobs timing out); primary evidence shows only documentation references to registry.yml and BRAIN_SOURCE within `skills/` and `TOOLS.md`.

Conclusion / Next steps:
- The duplicated `sekitoba` entries described in Issue #10 likely originate in the external `bunny_stack`/`sekitoba` repo (the external brain at $BRAIN_SOURCE) where registry.yml is located and where the list-generation code exists.
- To fully fix the bug we need access to `${BRAIN_SOURCE:-/Users/tensorinfo/source/bunny_stack}` repo and specifically the code that reads `registry.yml` and produces the quick-select list. Apply deduplication by project id at the parsing or rendering step and add unit tests there.

Logs:
- Branch created: issue-10/dedup-registry-projects
- Commands run: rg/sed/gh issue view (fetched Issue #10), multiple repo-wide searches (some timed out); see command history in shell for details.

Attempted edits: None (no target code found in this repo).

Reference: This change references Issue #10 and documents diagnostic discovery.
