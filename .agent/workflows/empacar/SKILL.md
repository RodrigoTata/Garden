---
name: empacar
description: Update README and docs to reflect the current state of the codebase, then stage changes for a manual commit. Use when the user wants to prepare a release, document a sprint's worth of work, or before pushing to a shared branch.
---

# Empacar (Pack & Document)

Synthesize everything that has changed since the last commit into human-readable documentation, then prepare a clean, commit-ready workspace. The goal is to leave the repo in a state where any developer — human or AI — can open it and understand what the system does and what changed.

> **Do NOT interview the user before starting.** Derive everything from the conversation context, git history, and codebase exploration. Only pause to ask if you find a genuine ambiguity that would change the output materially.

---

## Process

### Step 1 — Explore & Diff

Read the current state of the codebase. Understand what has changed since the last commit.

- Run `git status` to see which files are staged, modified, or untracked.
- Run `git diff HEAD` (or `git diff HEAD~1`) to understand the actual code changes.
- Scan the module directories listed in `README.md` and compare against what exists in `src/`.
- Read any `.agent/workflows/` files to understand ongoing automation or operational changes.
- Identify newly added features, bug fixes, architectural decisions, or configuration changes.

Use the project's domain language throughout — if the codebase calls something a "Dispensación", use that term. Don't normalize to generic names like "order" or "transaction."

---

### Step 2 — Update `README.md`

`README.md` is the **public face** of the repo. It must be accurate, concise, and useful to a developer seeing the project for the first time.

Follow the rules in [README-FORMAT.md](README-FORMAT.md). In summary:

- **Features section**: Add or update bullet points for any newly implemented feature. Remove outdated references.
- **Tech Stack section**: Update if any dependency was added, removed, or meaningfully upgraded.
- **Project Structure section**: Add any new `src/` modules that aren't listed. Remove any that no longer exist.
- **Endpoints section**: Add or adjust endpoint summaries if new routes were added.
- **Workflows section**: Mention any new `.agent/workflows/` skill if it was added in this session.

**Tone**: Written in English. Professional. No filler. No marketing language. Accurate to the real code.

---

### Step 3 — Update `docs.md`

`docs.md` is the **architecture guide** for maintainers. It should reflect the technical decisions made in this session.

Follow the rules in [DOCS-FORMAT.md](DOCS-FORMAT.md). In summary:

- **Architecture section**: Update if a new layer, pattern, or module boundary was introduced.
- **Design Patterns section**: Document any new pattern (e.g., Cron-based TTL, Strategy extension, Guard).
- **Data Flows section**: Add any new end-to-end flow (e.g., "Dispensation TTL Expiry Flow").
- **External Integrations section**: Update if a new service or SDK was integrated.
- **Data Model section**: Note any entity or schema changes (new columns, new relations, enum changes).

**Tone**: Written in English. Technical. Dense. Written for developers who already know the stack.

---

### Step 4 — Stage for Commit

After the documentation files are updated and saved:

1. Run `git status` to show the user a clean summary of what is about to be committed.
2. Suggest a **conventional commit message** following the format:

   ```
   <type>(<scope>): <short summary>

   <optional body>
   ```

   Where `type` is one of: `feat`, `fix`, `refactor`, `docs`, `chore`, `test`.  
   Where `scope` is the primary module affected (e.g., `dispensacion`, `store`, `cron`).

3. **Do NOT run `git add` or `git commit` automatically.** The commit is always the developer's decision. Only propose the command — never execute it.

Present the proposed commit message in a code block so the user can copy it directly.

---

## Output Contract

At the end of the skill, deliver:

1. **Updated `README.md`** — saved to disk, changes visible.
2. **Updated `docs.md`** — saved to disk, changes visible.
3. **`git status` snapshot** — so the developer sees exactly what files will be committed.
4. **Proposed `git commit` command** — ready to copy and paste.

---

## Guardrails

- **Do not remove accurate content.** Only remove content that is demonstrably wrong or references modules that no longer exist.
- **Do not over-document.** README and docs should be updated only for changes that are complete. In-progress or experimental changes should be left out.
- **Do not invent features.** Every line in the docs must be traceable to real code in the repo.
- **Do not commit.** Staging and committing is always manual. The skill only prepares.
