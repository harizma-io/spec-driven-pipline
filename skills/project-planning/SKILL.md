---
name: project-planning
description: |
  Plan new projects: adaptive interview, tech decisions,
  fill all project documentation (project-knowledge) in one session.

  Use when: "сделай описание проекта", "запиши описание проекта в документацию",
  "проведи со мной интервью для описания проекта", "заполни документацию проекта",
  "начни планирование проекта", "давай опишем проект", "plan a new project",
  "fill project documentation"

  For feature planning use user-spec-planning; for tech planning use tech-spec-planning.
---

# Project Planning

Conduct adaptive interview → make tech decisions → fill all project documentation in one session.

## Output Files

**Project Knowledge** (`.claude/skills/project-knowledge/references/`):
- **project.md** — overview, audience, problem, key features, scope
- **architecture.md** — tech stack, project structure, dependencies, data model
- **patterns.md** — git workflow (code patterns, testing, business rules are filled later during development)
- **deployment.md** — platform, environment, CI/CD, monitoring
- **ux-guidelines.md** — only if project has significant UI

## Interview Methodology

**One question at a time.** Ask one question, wait for the answer, then form the next question based on the response.

**Build on answers.** If user mentioned a domain — ask domain-relevant follow-ups. If they said something vague — clarify that specific point.

**Confirm understanding.** After 3-5 questions, briefly summarize what you understood. Catches misunderstandings early.

**Help when stuck.** When user says "not sure" or "don't know":
1. Say it's OK
2. Offer 2-3 common approaches for their type of project
3. Ask which is closer
4. If still uncertain and optional — mark TBD, move on
5. If still uncertain and required — break into simpler sub-questions

**Recount on scope changes.** If user suddenly adds many features or reveals unexpected complexity — stop and recount total scope. Show the updated list, confirm you understood correctly.

**If code exists.** Analyze the codebase in parallel with the interview:
- Scan package.json / requirements.txt / go.mod for tech stack
- Check directory structure for architecture patterns
- Read configs for deployment setup
- Use discoveries to ask more targeted questions and pre-fill technical decisions

## Phase 1: Project Discovery

### 1.1 Verify Environment

```bash
test -d .claude/skills/project-knowledge/references && echo "HAS_PK" || echo "NO_PK"
test -f CLAUDE.md && echo "HAS_CLAUDE_MD" || echo "NO_CLAUDE_MD"
```

- `NO_PK` or `NO_CLAUDE_MD` → tell user to run `/init-project` first
- All clear → proceed

### 1.2 Check for Existing Code

```bash
ls -d src/ lib/ app/ cmd/ pkg/ *.py *.ts *.js *.go Cargo.toml 2>/dev/null | head -5
```

Code found → note for Phase 2 (extract stack from code instead of deciding from scratch).

### 1.3 Interview

Ask user to describe the project in free form. Let them say as much or as little as they want.

Then ask adaptive questions to cover three areas:

**Project Overview:**
- What the project does (one-line + context)
- Who uses it and why (target audience + use case)
- What problem it solves (core pain point)
- 3-5 key features (high-level only)
- What it explicitly doesn't do (out of scope)

**Features & MVP:**
- Key features with descriptions
- What's included in MVP (launch scope)
- What comes later (post-launch ideas) — note these for the backlog
- Priority for each: Critical / Important / Nice-to-have

**Development Approach:**
- All at once or phased?
- If phased: how to group features, what's MVP
- If migration: current system, data migration, risks, rollback plan

### 1.4 Checkpoint

Move to Phase 2 when you can:
- Write a clear, non-vague project.md
- List key features with priorities and MVP scope
- Describe the development approach

Not every answer needs to be perfect. TBD is acceptable for optional aspects.

## Phase 2: Technical Decisions

### 2.1 New Project (no code)

1. **Propose tech stack** based on Phase 1: frontend, backend, database, key dependencies
2. **Verify via Context7:** resolve-library-id → query-docs for each technology. Update if Context7 reveals deprecations or better alternatives. If Context7 unavailable — proceed and note it.
3. **Propose deployment:** platform, CI/CD approach, environments
4. **Show proposal:**

   ```
   Предлагаю tech stack:
   **Frontend:** [technology] — [why]
   **Backend:** [technology] — [why]
   **Database:** [technology] — [why]
   **Deployment:** [platform] — [why]

   Согласен или есть правки?
   ```

5. Iterate until user approves.

### 2.2 Existing Code

1. **Extract stack** from package files, configs, directory structure
2. **Verify via Context7:** check versions and best practices
3. **Confirm with user:** show what you found, ask about gaps (deployment, missing pieces)
4. Iterate until confirmed.

### 2.3 Checkpoint

Tech stack and deployment approach approved by user.

## Phase 3: Fill Documentation

Documentation goal: someone opens these files and understands the project without reading code. Describe what exists, what it does, and why. Record decisions, operational details (server addresses, deploy procedures, log locations), high-level component overview. Skip what's obvious from code. No code blocks or pseudocode — link to source files. No duplication between files.

Use Edit tool to replace template placeholders with real content. Content language: English.

### 3.1 Project Knowledge Files

**project.md** — from Phase 1 interview:
- Project overview, target audience, core problem
- Key features with priorities and MVP scope
- Post-launch ideas (if discussed)
- Out of scope

**architecture.md** — from Phase 2 decisions + codebase analysis:
- Tech stack with "why" for each choice
- Project structure (directory tree)
- Key dependencies (only critical ones, not everything)
- External integrations
- Data flow
- Data model (fill if known, leave template sections if TBD)

**patterns.md** — fill git workflow section:
- Branch structure, branch decision criteria
- Testing requirements per branch
- Security gates (pre-commit, pre-push)
- Leave code patterns, testing methods, and business rules sections minimal — filled during development as patterns emerge

**deployment.md** — from Phase 2 decisions:
- Platform, type, rationale
- Deployment triggers (what deploys where)
- Environments and URLs
- Environment variables (reference .env.example)
- Monitoring: fill if configured, note "not yet configured" if not

**ux-guidelines.md** — only if project has significant UI. Skip entirely for CLIs, APIs, bots without custom UI.

### 3.2 Backlog (if applicable)

If user discussed post-launch features during the interview, save them to a backlog file.

1. Check CLAUDE.md for a `Backlog:` path — use it if present
2. If no backlog path found — ask user where to create it (e.g., `docs/backlog.md`, `BACKLOG.md`)
3. After creating the backlog file, add `Backlog: <path>` to CLAUDE.md so it's not lost
4. If user declines — skip, the information is already captured in project.md

### 3.3 Checkpoint

All output files from "Output Files" section created. No template placeholders remain.

## Phase 4: Review & Commit

### 4.0 Self-Verify

Before presenting to user, verify:
- All output files listed in "Output Files" section were created
- No template placeholders remain (search for `[Project Name]`, `[Description]`, etc.)
- project.md contains all key features discussed in interview
- architecture.md tech stack matches user-approved decisions from Phase 2

If any check fails — fix before proceeding.

### 4.1 Documentation Review

Run `documentation-reviewer` agent (Task tool, sonnet) on the project. Fix critical and major findings. Minor findings — fix or leave at your discretion.

### 4.2 Show Files

Tell user (Russian):

```
Документация заполнена:

**Project Knowledge:**
- [project.md](.claude/skills/project-knowledge/references/project.md) — описание проекта
- [architecture.md](.claude/skills/project-knowledge/references/architecture.md) — архитектура и стек
- [patterns.md](.claude/skills/project-knowledge/references/patterns.md) — git workflow
- [deployment.md](.claude/skills/project-knowledge/references/deployment.md) — деплой

Посмотри. Всё правильно? Есть что изменить?
```

Include ux-guidelines.md and backlog file in the list if they were created.

### 4.3 Iterate

- Changes requested → edit files → show updated list → repeat
- Questions → answer → continue waiting for approval
- Repeat until user approves

### 4.4 Commit

After approval, ask: "Закоммитить?"

If yes — commit all created documentation files:
```bash
git add .claude/skills/project-knowledge/references/*.md

git commit -m "$(cat <<'EOF'
feat: fill project documentation

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

Include backlog file in git add if it was created.

Final message: "Документация заполнена! Можно начинать разработку."
