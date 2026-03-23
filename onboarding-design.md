# AI Workflow Onboarding for Principal Engineer

**Date:** 2026-03-17
**Author:** Matvii Sakhnenko
**Duration:** 2 half-days (~3-4 hours each, flexible)
**Format:** Guided setup and discovery — you drive on Day 1 (they follow along on their machine), they drive on Day 2 (you navigate)
**Audience:** Principal engineer joining the iOS team. Strong iOS background, uses Claude/Codex at surface level but hasn't tapped into skills, subagents, AGENTS.md context, hooks, MCP servers, or orchestration workflows.

---

## Prerequisites

Before Day 1, ensure the following are ready on their machine:

### Must Have
- [ ] iOS repo cloned and accessible (`~/Developer/Finn/ios-app` or equivalent)
- [ ] Xcode installed with iOS simulators
- [ ] Admin rights to install apps (Homebrew, CLI tools)
- [ ] GitHub account with repo access and `gh` CLI authenticated
- [ ] Google account with Firebase project access (for Firebase MCP auth)
- [ ] Atlassian credentials (for Jira/Confluence MCP)
- [ ] Node.js installed (required by several MCP servers — `brew install node` if missing)

### Nice to Have
- [ ] Figma account with access to FINN design files
- [ ] Amplitude account access
- [ ] A candidate backlog item for Day 2 (low-risk, <4 story points, no Figma dependency required)

---

## Day 1: The AI Workstation (~3-4h)

Goal: Transform their Mac from "I use Claude sometimes" to a fully integrated AI-native development environment.

**Keyboard:** You drive and demonstrate, they follow along setting up on their own machine.

### Block 1: Foundation (~50min)

**Goal:** Get their machine speaking AI fluently.

*Budget extra time here — first-time installs and auth flows can be unpredictable.*

#### Warp Terminal
- Install and configure Warp
- Show advantages for AI CLI work: command blocks, AI suggestions, searchable history
- Configure as default terminal

#### Wispr Flow
- Install and set up voice-to-prompt
- Demonstrate the daily workflow: prompting Claude Code by voice while reading code
- Show how it reduces context-switching between thinking and typing

#### Claude Code Basics
- Install and authenticate
- Shortcuts most users don't know:
  - `!<command>` — execute bash instantly, inject output into context
  - `/btw` — ask a side question while Claude is working
  - `Esc Esc` — rewind to clean checkpoint
  - `Shift+Tab` — switch between Auto-Accept, Plan, and Normal mode
  - `Ctrl+S` — save draft (like git stash for prompts)
  - `--continue` / `--resume` — session recovery
  - `/vim` — vim mode for editing prompts
  - Thinking modes (adaptive reasoning): `think`, `think hard`, `ultrathink`
  - `/context` — see token usage breakdown (system prompt, MCP servers, memory)

### Block 2: Context — How the Repo Teaches AI (~50min)

**Goal:** The "aha moment" — show how project context transforms Claude from a generic assistant into a team member who knows the codebase.

#### CLAUDE.md / AGENTS.md Walkthrough
- Open the iOS repo's `CLAUDE.md` (repo root) and `AGENTS.md`
- Walk through each section: project overview, architecture, module structure, engineering policies, concurrency settings, design system rules
- **What context enables and how to shape it:** Walk through the three layers of context (CLAUDE.md/AGENTS.md → Rules → Module AGENTS.md) and show how each one changes Claude's behavior. Focus on how to improve context over time: wrong answer twice → add a rule; new architecture decision → update AGENTS.md; repeating instructions → create a skill.

#### Rules
- Show `.claude/rules/` directory
- Walk through `unleash-cli.md` as an example of how rules shape Claude's output formatting
- Show `module-agents.md` and how module-level AGENTS files provide domain-specific patterns

#### Skills Catalog
- Browse the full skill catalog from AGENTS.md
- Show how skills are invoked: `/ios-review-code`, `/ios-pre-push`, `/ios-write-unit-test`
- **Live invocation:** Run one skill on existing code (e.g., `/ios-review-code` on a recent file) to demonstrate the difference between "asking Claude to review" vs. "invoking a structured review skill"

#### Documentation Index
- Show how AGENTS.md links to canonical docs (architecture, localization, CI workflows, accessibility)
- Explain the principle: Claude finds the right answer in project docs instead of hallucinating

#### Why This Matters (Philosophy Discussion)
Take a few minutes to discuss the *why* — this is what separates surface-level AI usage from a real workflow:
- **Context is the multiplier.** The same model with project context produces 10x better answers than without. CLAUDE.md/AGENTS.md is the highest-leverage investment.
- **Skills encode team knowledge.** Instead of every engineer writing their own prompts, skills capture the best approach once and make it repeatable.
- **When NOT to use AI.** Recognize when manual work is faster — quick one-line fixes, deeply familiar code, creative architectural decisions that need human judgment.
- **Token awareness.** Show `/context` to see what's being loaded. Understand that thinking modes (`ultrathink`) cost more but help with complex reasoning. MCP servers add context overhead.

### Block 3: Orchestration (~50min)

**Goal:** Claude Code as conductor, Codex as task manager. This is the power-user layer.

#### Claude Code Power Features
- **Subagents:** Dispatch parallel research/implementation tasks. Show `Agent` tool dispatching independent work.
- **Hooks:** Automated behaviors triggered by events (e.g., pre-commit checks, post-tool-call actions). Show how to configure in `settings.json`.
- **Plan Mode:** `Shift+Tab` to toggle. Show how plan mode lets Claude think through an approach before executing.
- **Session management:** Named sessions, `--continue`, `--resume`. Show how to maintain continuity across work sessions.

#### Codex Native App
- Install/set up if not already present
- Worktree management — show how Codex creates isolated git worktrees for parallel work
- Task dispatching — send tasks to Codex, review results
- How Codex and Claude Code complement each other: Claude Code for interactive deep work, Codex for async/parallel task execution

#### How We Work With AI (Conceptual)
- Explain the two layers:
  - **Superpowers skills** (process layer) — brainstorming → writing-plans → subagent-DD → finishing. For complex, multi-step features.
  - **Repo skills** (domain layer) — `/ios-review-code`, `/ios-pre-push`, `/ios-write-unit-test`, `/ios-write-snapshot-test`, `/ios-pr`. For daily tasks.
- Most day-to-day work uses repo skills directly. The full Superpowers pipeline is for when you need Claude to think before it acts.
- For ported features, `/ios-port-from-android` generates a reference doc first.
- *Note: The design-to-code step uses Figma MCP, which we'll set up in the next block.*

### Block 4: Integrations (~50min)

**Goal:** Connect the external world to the AI workflow.

*Budget extra time — Firebase auth and MCP server setup can have friction.*

#### MCP Servers Setup
Set up each MCP server on their machine:

| MCP Server | Purpose | Setup |
|------------|---------|-------|
| **XcodeBuildMCP** | Build, test, run from Claude | `claude mcp add -s user XcodeBuildMCP npx xcodebuildmcp@latest -e XCODEBUILDMCP_DYNAMIC_TOOLS=true` |
| **Firebase** | Crashlytics, Remote Config, analytics | `./Scripts/setup-firebase-mcp.sh` (requires Google auth — budget 5-10min for first-time OAuth) |
| **Amplitude** | Analytics insights | `claude mcp add -t http -s user Amplitude "https://mcp.eu.amplitude.com/mcp"` |
| **Atlassian** | Jira issues, Confluence pages | `claude mcp add -t sse -s user Atlassian https://mcp.atlassian.com/v1/sse` |
| **Figma** | Design-to-code, component inspection | Configured as Claude Code plugin — verify with `/mcp` and test with a known Figma file URL |

- For each: install, authenticate, run a quick test query
- Show practical examples: "Show me the latest Crashlytics crashes for iOS", pulling a Figma frame, checking a Jira ticket

#### Quick Wins Demo
- Ask Claude to build a scheme via XcodeBuildMCP
- Pull an existing Figma design and show the design-to-code flow (this connects back to the pipeline from Block 3)
- Query Amplitude for a real metric

#### Security Awareness (5min)
Quick but important ground rules:
- **Never paste secrets, API keys, or PII into prompts.** Claude Code reads files — point it at the file instead of copying contents.
- **MCP auth tokens** are stored locally in Claude Code's config. They don't leave the machine.
- **Always review AI-generated code** before committing — especially around auth, networking, and data handling. The `/ios-review-code` skill helps here.

---

## Day 2: The AI Workflow (~3-4h)

Goal: Go from setup to muscle memory. Hands-on, they drive.

**Keyboard:** They drive, you navigate. Resist the urge to take over.

### Block 1: Warm-up — Guided Skill Runs (~50min)

**Goal:** Build confidence with individual skills before chaining them.

Exercises (they drive, you navigate):

1. **Code Review** — Pick any recently changed file, run `/ios-review-code`. Discuss the output, show how it checks for Warp tokens, accessibility, architecture, and repo policy.

2. **Pre-push Validation** — Check out a branch with changes, run `/ios-pre-push`. Show lint, test, and snapshot validation.

3. **Unit Test Generation** — Pick an existing ViewModel or UseCase, run `/ios-write-unit-test`. Show how it follows Swift Testing patterns from the repo.

4. **Snapshot Test** — Pick an existing SwiftUI view, run `/ios-write-snapshot-test`. Show how it generates accessibility snapshot coverage.

Optional quick wins depending on interest:
- `/ios-warp-design-system` — look up Warp tokens for a hardcoded value
- `/ios-localization-flow` — run localization + SwiftGen workflow
- `/ios-ios26-compatibility-check` — validate Liquid Glass gating on a view

### Block 2: End-to-End Development (~2-2.5h)

**Goal:** Full AI-driven development flow on a real (or realistic) task.

#### Choosing the Task

**Use Option A (real backlog item) if:**
- There's a low-risk item <4 story points with clear acceptance criteria
- It doesn't require a Figma design (or one already exists)
- It touches code they'll work on anyway — makes the exercise immediately valuable

**Use Option B (realistic exercise) if:**
- No good backlog items available, or they're too risky for a learning session
- You want a controlled scope that guarantees hitting all pipeline stages

#### Option A: Real Backlog Item
1. Describe the task in Claude Code (it already has AGENTS.md context)
2. Use Plan mode (`Shift+Tab`) for complex tasks — let Claude think first
3. Execute — Claude implements using repo conventions
4. `/ios-review-code` — review the changes
5. `/ios-pre-push` — validate before push
6. `/ios-pr` — create the PR

#### Option B: Realistic Exercise
Scoped exercises that don't need Figma:

- **"Add a feature flag to an existing screen"** — touches Unleash CLI, SwiftUI view modification, and the full validation pipeline. Realistic scope, no Figma needed.
- **"Port a small Android behavior to iOS"** — demonstrates `/ios-port-from-android` skill. Pick a simple Android PR/branch as source.
- **"Write tests for an untested ViewModel"** — demonstrates test generation skills and shows how Claude understands the existing architecture.

Either way, the flow follows the same pipeline stages. The goal is experiencing the full loop, not necessarily shipping the result.

#### Debugging & Course-Correction
When things go wrong (and they will — that's part of learning):
- **Claude gives a wrong answer:** Show how to `Esc Esc` rewind and rephrase, or add context ("look at the existing pattern in X file")
- **MCP server fails:** Show `/mcp` to check status, restart if needed. This is normal.
- **Skill produces unexpected output:** Show how to read the skill source (`.claude/skills/`) to understand what it's doing, and adjust the prompt
- **Build fails after AI changes:** Show how the build-fixer agent or `/ios-pre-push` can catch and fix issues

### Block 3: Wrap-up & Customization (~40min)

**Goal:** Make it stick, and show them how to extend the system.

#### Reference Materials
- Point to `Documentation/CLAUDE-CODE-CHEAT-SHEET.md` as their daily reference
- Point to AGENTS.md skill catalog — they can discover new skills as needs arise
- Show how to find and install new skills (`/find-skills`)

#### Skill Authorship (10min)
A principal engineer should know how to extend the system, not just use it:
- Show the anatomy of a skill: `.claude/skills/<skill-name>/SKILL.md`
- Walk through one simple skill file to show the structure (name, description, trigger conditions, instructions)
- Explain how the skill `name` field auto-creates the slash command
- Show the AGENTS contribution pipeline: classify as policy/workflow/reference, keep AGENTS concise, run `/ios-agent-docs-hygiene`
- Encourage: "If you find yourself repeating a workflow, turn it into a skill"

#### Reflection
- What resonated most? What will they use daily?
- What felt like overkill for their workflow?
- Any tools/workflows they want to explore further?

#### Ongoing Support
- They can always `--resume` a session to pick up where they left off
- Encourage them to customize: add their own rules, modify hooks, contribute skills back to the repo
- Offer to pair on their first solo AI-driven feature if they want backup

---

## Tool Inventory Checklist

Everything that should be installed and configured by end of Day 1:

### Apps & CLI Tools
- [ ] Warp Terminal (configured as default)
- [ ] Wispr Flow (voice-to-prompt)
- [ ] Claude Code CLI (authenticated)
- [ ] Codex native app (authenticated, worktrees configured)

### MCP Servers
- [ ] XcodeBuildMCP (build/test/run from Claude)
- [ ] Firebase MCP (Crashlytics, Remote Config — Google auth completed)
- [ ] Amplitude MCP (analytics)
- [ ] Atlassian MCP (Jira/Confluence)
- [ ] Figma MCP (verified working via `/mcp`)

### IDE (optional, if using VSCode)
- [ ] VSCode + SweetPad extension
- [ ] Copilot + Copilot Chat (license from ServiceNow)
- [ ] Build server config (`./Scripts/generate-build-server-config.sh`)

### Repo Context
- [ ] CLAUDE.md / AGENTS.md understood
- [ ] Skills catalog browsed
- [ ] At least one skill invoked successfully
- [ ] At least one MCP query executed successfully

---

## Success Criteria

By end of Day 2, the principal engineer should be able to:

1. **Start a feature** by describing it in Claude Code with repo context, using Plan mode for complex tasks
2. **Invoke skills** for code review, pre-push, test generation — using AGENTS.md as reference if needed, but without hand-holding
3. **Use MCP servers** to query Crashlytics, Jira, or Figma from Claude Code
4. **Dispatch subagents** for parallel work or research tasks
5. **Use Codex** for worktree-based async task execution
6. **Prompt effectively** — know when to use thinking modes, plan mode, voice input, and when AI isn't the right tool
7. **Navigate the repo context** — know where AGENTS.md, skills, rules, and docs live and how they shape Claude's behavior
8. **Extend the system** — understand skill anatomy well enough to create or modify one when needed
