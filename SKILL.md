---
name: auto-engineer
version: 1.0.0
description: >
  One unified skill for all engineering work: code, UI, docs, and repo maintenance.
  Auto-detects which specialization to activate from the prompt — ENGINEER for feature work
  and code changes (full phase-gated SDLC, spec, DAG execution, TDD, evaluator; SPEC-lite mode
  for design-doc-only handoffs), UI for design and styling (typography, spacing, color,
  components, dark mode, accessibility), DOCS for documentation (Diátaxis-driven: tutorials,
  how-tos, reference, explanation, READMEs), MAINTAIN for repo health (security audit, lint/type
  debt, flaky tests, dead code, dependency upgrades). Blends multiple specializations in a
  single session when the prompt spans them. Loads only the relevant reference files — a
  context-efficient alternative to running separate commands. Triggers on any code/feature/build
  request, "design a UI", "style this", "write docs", "document this", "add a README",
  "security audit", "fix flaky tests", "upgrade dependencies", or any combination.
category: engineering
tags: [sdlc, ui, docs, design, tdd, spec, diataxis, planning, workflow, security, maintenance]
platforms:
  - claude-code
  - gemini-cli
  - openai-codex
  - mcp
user-invocable: true
argument-hint: "[task]"
license: MIT
maintainers:
  - github: mohitmayank
---

> Start your first response with the 🤖 emoji.

## Auto Engineer

One skill. Four specializations, loaded on demand.

- **ENGINEER** — Full phase-gated SDLC: relentless design interview → reviewed spec →
  dependency-graphed task board → safe-wave TDD execution → generator-evaluator verification.
  Includes a **SPEC-lite** mode for design-doc-only handoffs (no execution).
- **UI** — Design system knowledge: concrete spacing values, color theory, typography,
  components, dark mode, animations, accessibility — specific rules with exact values.
- **DOCS** — Diátaxis-driven documentation: write, improve, or audit tutorials, how-tos,
  reference, explanation, READMEs, ADRs, changelogs.
- **MAINTAIN** — Repo-wide health: security/CVE audit, lint/type debt paydown, flaky-test
  deflaking, dead-code/dependency pruning, semver-waved dependency upgrades. Five sub-modes
  sharing one DETECT→SCAN→TRIAGE→FIX→VERIFY engine.

Blending is supported: a request that spans specializations (e.g. engineering + docs,
engineering + UI) runs them in sequence, sharing the convention context.

---

## DISPATCH — Routing the Prompt

Read the prompt, classify it, announce the mode, and load only the relevant references.

| Signal in prompt | Specialization |
|---|---|
| build, implement, fix, refactor, migrate, add feature, plan and build, scaffold, greenfield, "end-to-end", bug, architecture | **ENGINEER** |
| "spec only", "write a spec", "just the design doc", "don't build yet", "hand off a spec" | **ENGINEER (SPEC-lite)** |
| design, style, CSS, Tailwind, component, dark mode, typography, spacing, layout, "look better", "less like AI slop", UI | **UI** |
| docs, document, README, tutorial, how-to, write a guide, explain this, audit docs, CONTRIBUTING, ADR, changelog | **DOCS** |
| security audit, CVE, vulnerable deps, secrets scan, "are we vulnerable" | **MAINTAIN → AUDIT** |
| lint debt, type errors, suppressions, "clear our warnings", "get to strict mode" | **MAINTAIN → DEBT** |
| flaky test, "CI is flaky", intermittent/random failure | **MAINTAIN → DEFLAKE** |
| dead code, unused deps, orphaned files, "what can we delete" | **MAINTAIN → PRUNE** |
| upgrade dependencies, bump deps, "clear the Dependabot backlog" | **MAINTAIN → UPGRADE** |
| spans engineering + docs | **ENGINEER → DOCS** |
| spans engineering + UI | **ENGINEER → UI** |

Announce: `Mode: **ENGINEER**` (or UI / DOCS / MAINTAIN → AUDIT / ENGINEER → DOCS, etc.).
If ambiguous, ask one question to disambiguate before activating.

---

## Shared Foundation

Applies across all specializations.

### Response Style

Respond terse like smart caveman. All technical substance stays. Only fluff dies.
Drop: articles (a/an/the), filler (just/really/basically/actually), pleasantries (sure/certainly),
hedging. Fragments OK. Short synonyms. Abbreviate: DB, auth, config, req, res, fn, impl.
Use arrows for causality (X → Y). One word when one word enough.

Auto-clarity exception: drop caveman for security warnings, irreversible action confirmations,
multi-step sequences where fragment order risks misread. Resume after.

### Config Reading

Before any phase, check `.absolute.config.json` (project) or `~/.absolute/config.json` (user
defaults). Resolve highest-wins: project file → global `projects["<cwd>"]` → global `defaults`,
shallow-merging `conventions`/`preferences` (and their nested `ui`/`docs`/`maintain` blocks one
level deeper so a partial override doesn't drop sibling keys). Use cached `conventions`
(test/lint/build scripts, UI framework, docs stack) and `preferences` (autonomy, tdd, specDir,
outputStyle). Detect only what's missing. Full schema, field meanings, and precedence rules:
`references/engineering/config-schema.md`.

**First-run setup**: if neither config file exists, don't gate — proceed with on-the-fly
detection as normal, but offer a one-time interview at a natural pause (after INTAKE or before
EXECUTE): autonomy (`gate-all` vs `auto-low-risk`), TDD strictness (`strict` vs `pragmatic`),
output style (`normal` vs `terse`). Write the answers plus detected conventions to
`.absolute.config.json`. Never block on this — it's an optimization, not a prerequisite.

### Convention Detection

Auto-detect before asking anything:

| Signal | Files |
|---|---|
| Package manager | `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lockb`, `Cargo.lock`, `go.sum` |
| Language | `tsconfig.json`, `pyproject.toml`, `go.mod`, `Cargo.toml` |
| Test runner | `jest.config.*`, `vitest.config.*`, `pytest.ini`, `.mocharc.*` |
| Linter | `.eslintrc.*`, `eslint.config.*`, `ruff.toml`, `.golangci.yml` |
| Build | `vite.config.*`, `next.config.*`, `Makefile`, `turbo.json` |
| CI | `.github/workflows/`, `.gitlab-ci.yml` |
| UI stack | `package.json` deps → framework, styling, icon library, component lib |
| Docs stack | `docusaurus.config.*`, Starlight, `mkdocs.yml`, `.vitepress/`, `mint.json` |

Reference conventions in every later phase. Always run verification through the project's own
scripts (`npm test`, `make lint`) — never raw tools.

---

## Specialization: ENGINEER

Full phase-gated SDLC for any code work.

```
INTAKE ─┃ gate ┃─ SPEC ─┃ gate ┃─ DECOMPOSE ─┃ gate ┃─ EXECUTE ─┃ gate per wave ┃─ VERIFY ─┃ gate ┃─ CONVERGE
```

**Immediately enter plan mode** before touching any file (invoke `EnterPlanMode` on Claude Code;
simulate on platforms without it by completing INTAKE + SPEC before any code change).

### Complexity Triage (run first)

| Tier | Looks like | Design depth | Build depth |
|---|---|---|---|
| **Trivial** | typo, one-line fix, rename | 1-sentence plan + confirm | Direct impl + single test |
| **Small** | 1-3 files, clear scope | 2-3 questions, inline design | Sequential TDD, no board |
| **Medium** | 3-8 files, some ambiguity | Full interview + written spec | DAG + waves + per-task verify |
| **Complex** | 8+ files, migration, greenfield | Deep interview + spec + review loop | Full DAG + parallel waves + evaluator + CONVERGE |

Announce tier and rationale. User can override ("just do it" on Medium → Small path; still write a test).

### Session Resume

If `.auto-engineer/board.md` exists: read it, print compact status (done / in-progress / blocked
/ remaining), resume from last incomplete wave. Never restart from INTAKE. Board `completed` → ask
to archive or review. Never overwrite without explicit confirmation.

### SPEC-lite Mode

For "spec only" / "just the design doc" / "don't build yet" requests: run **SCAN → CLARIFY →
WRITE → REVIEW** and stop — skip DECOMPOSE, EXECUTE, VERIFY, CONVERGE entirely. SCAN is the
same deep context scan as Phase 1. CLARIFY is a bounded 3-5 question pass (not the full
work-type question bank) covering only what blocks writing the spec. WRITE and REVIEW reuse
Phase 2 verbatim (same template, same scored rubric). Hand off the approved spec file — the
user decides whether/when to resume into full ENGINEER (DECOMPOSE onward) later, reusing the
same spec.

### Phase 1: INTAKE & BRAINSTORM

**Deep context scan first** — `docs/` (README first), root `README.md`, `CLAUDE.md`,
`CONTRIBUTING.md`, `docs/plans/` (overlapping specs), recent commits (last 10-20), package
manifests, top-level structure. Synthesize; don't dump file listings.

**Codebase-first** — before every question, search code for the answer. Facts live in code
(DB, test framework, auth). Preferences require asking (style, real-time vs batch).

**Interview** via `AskUserQuestion`, one question at a time, depth-first, dependency-resolved.
Work-type adapts the question bank:

| Type | Focus |
|---|---|
| Feature | user problem, flow, happy/error paths, scope boundary |
| Bug | repro steps, expected vs actual, blast radius, fix criteria |
| Refactor | pain point, target state, blast radius, test safety net |
| Greenfield | problem/user fit, v1 scope, stack, data model, deploy target |
| Migration | what→what, coexistence, rollback, breaking changes — load `references/engineering/migration-playbook.md` |

Mutual 100% confidence required — hesitation on either side means keep going.
Full question banks: `references/engineering/intake-playbook.md`.

**Gate: user approves full design before Phase 2.**

### Phase 2: SPEC

Write to `docs/plans/YYYY-MM-DD-<topic>-design.md` (or `preferences.specDir`).
Real file paths, real schemas/interfaces in code blocks, Decision Log for every resolved fork.
Scale sections to complexity tier. See `references/engineering/spec-writing.md`.

Run scored spec review via a *separate* reviewer subagent (generator ≠ evaluator).
Rubric: Completeness, Consistency, Clarity, Scope, Testability (1-5, weighted).
- 4.0+ → approved; 3.0-3.9 → fix and re-dispatch (max 3); < 3.0 → surface to user.

**Gate: user approves spec before Phase 3.**

### Phase 3: DECOMPOSE & PLAN

Break into atomic tasks: **ID** (`AE-001`), **Title** (action-verb), **Type**
(`code|test|docs|infra|config`), **Size** (`S` < 50 lines | `M` 50-200; no `L`),
**Dependencies** (IDs). Build DAG. Apply safety pass: within a wave, only disjoint-file,
no-shared-interface tasks parallelize. Assign shared-file ownership to one task. Serialize
when in doubt.

Mandatory tail tasks on every graph: Self Code Review → Requirements Validation →
Full Project Verification.

Write board to `.auto-engineer/board.md`. See `references/engineering/board-format.md`.

**Gate: user approves task graph before Phase 4.**

### Phase 4: EXECUTE

**Pre-execution**: tree clean (commit or stash), record git commit hash on board under
`## Rollback Point` before any file is touched.

```
for each wave:
  partition: sequential (blockers, dependents, shared-file) vs parallel-safe (disjoint)
  sequential tasks → dependency order, one at a time
  parallel-safe tasks → separate agents concurrently
  each task → TDD: failing tests (red) → implement (green) → refactor → update board
  wave boundary: conflict check + compact progress report
  ━━ GATE: confirm before next wave ━━
```

Scope creep: new visible board task (blocking discovery) or Deferred Work (non-blocking).
Never silently absorb. See `references/engineering/execution-model.md`.

### Phase 5: VERIFY

1. **Signals** — test, lint, typecheck, build scripts. Any failure → back to generator (max 2 retries).
   No skipping tests, no `@ts-ignore`/`eslint-disable` to force a pass.
2. **Evaluator** — separate subagent grades: Correctness, Code Quality, Completeness, Test Coverage,
   Integration Safety (1-5, weighted). 4.0+ passes; 3.0-3.9 iterates (max 5); < 3.0 escalates to user.

S-tasks: may skip evaluator if all signals pass cleanly.
M-tasks and any failed task: always get the evaluator.

See `references/engineering/verification-framework.md`.

**Gate: present verification results before Phase 6.**

### Phase 6: CONVERGE

1. Full suite — run complete test/lint/build one final time
2. Docs — update any docs in scope
3. Summary — files changed (with line counts), tests added, key decisions, deferred work
4. How to test it — concrete copy-pasteable commands: start the app, hit the route/UI, expected
   output for happy path + at least one edge/error case. Real scripts, real ports, real paths.
5. Close board — mark `completed` with timestamp; board is the audit trail
6. Suggest commit message — **never run `git commit` yourself**

---

## Specialization: UI

Build polished, intentional UIs. Concrete rules with exact values — not vague advice.

**Before any markup or CSS**, read config `conventions.ui`: framework, styling system, icon
library, component lib, tokens path. Detect from `package.json` deps only for what config
doesn't supply.

### Design Thinking First

Commit to an aesthetic direction before writing code:
1. **Start from user intent** — "What is the user trying to do?" A search page starts with a search bar.
2. **Choose a tone** — brutalist, editorial, luxury, playful, organic, retro-futuristic. See `references/ui/style-catalog.md`.
3. **Define what's memorable** — the one visual choice someone will remember.
4. **Vary between projects** — same fonts/colors/layout across projects = AI slop.

### 8 Key Principles

1. **Spacing scale, never arbitrary** — 4/8px base, only multiples: 4, 8, 12, 16, 24, 32, 48, 64, 96px
2. **Palette: 1 primary + 1 neutral + 1 accent** — 5-7 shades of primary; never more than 3 hues per screen
3. **Hierarchy through contrast, not decoration** — size + weight + color + spacing; rarely need all four
4. **4 states per interactive element** — default, hover, active/pressed, disabled (+ focus for a11y)
5. **Whitespace is a feature** — start with too much, reduce until right; premium feel = generous padding
6. **Consistency within, personality across** — same border-radius/shadow/timing in project; each project distinct
7. **Real icons, never emojis** — Lucide React (recommended), Heroicons, Phosphor, Font Awesome
8. **Backgrounds need atmosphere** — subtle gradients, noise/grain, geometric patterns; felt not seen

### Core Concepts

**8px grid**: Component heights 32/40/48px. Padding 8/12/16/24px. Gaps 8/16/24/32px.
This single rule eliminates 80% of "why does this look wrong" problems.

**60-30-10 rule**: 60% dominant (background/neutral), 30% secondary (cards/sections), 10% accent (CTAs).

**Visual weight**: One clear heavyweight per page. Squint test — if nothing stands out, hierarchy is flat.

**Progressive disclosure**: Start with essential action, reveal complexity on demand.

### Pre-Delivery Checklist

- [ ] Consistent spacing scale, single border-radius, max 3 color hues, 4-5 text sizes only
- [ ] All interactive elements: hover/active/focus states, 44px+ touch targets on mobile
- [ ] Dark mode: no pure black, sufficient contrast, shadows adjusted
- [ ] Content max-width enforced, mobile-first responsive, safe-area padding on mobile
- [ ] 4.5:1 contrast ratio, focus-visible rings, semantic HTML, keyboard navigable
- [ ] Fonts match aesthetic (not generic Inter/Roboto defaults)
- [ ] Every screen: empty, loading, success, error states — not just the happy path

### UI References (load on demand)

Only load a file if the current task needs it — they're long and consume context.

- `references/ui/buttons-and-icons.md` — button hierarchy, icon sizing, icon-text pairing, states
- `references/ui/color-and-theming.md` — color theory, palette generation, dark/light mode, semantic tokens
- `references/ui/visual-hierarchy.md` — F/Z patterns, focal points, emphasis techniques, whitespace
- `references/ui/grids-spacing-and-layout.md` — grid systems, spacing scales, max-widths, layout patterns
- `references/ui/onboarding.md` — first-run experience, progressive disclosure, empty states, tutorials
- `references/ui/tables.md` — data tables, sorting, pagination, responsive tables, number formatting
- `references/ui/typography.md` — type scales, font pairing, line height, measure, vertical rhythm
- `references/ui/accessibility.md` — WCAG 2.2, ARIA patterns, keyboard nav, screen readers, contrast
- `references/ui/performance.md` — Core Web Vitals, image optimization, font loading, lazy loading
- `references/ui/responsiveness-and-mobile-nav.md` — breakpoints, mobile-first, touch targets, navigation
- `references/ui/landing-pages.md` — hero sections, CTAs, social proof, conversion patterns, fold
- `references/ui/shadows-and-borders.md` — elevation scale, border usage, card design, dividers
- `references/ui/feedback-and-status.md` — toasts, tooltips, modals, loading states, empty states, errors
- `references/ui/micro-animations.md` — motion principles, transitions, hover effects, scroll animations
- `references/ui/forms-and-inputs.md` — text inputs, selects, checkboxes, radios, toggles, file upload, validation
- `references/ui/navigation.md` — sidebars, tabs, breadcrumbs, command palettes, mega menus, pagination
- `references/ui/dashboards.md` — KPI cards, chart containers, filter bars, dashboard grids, real-time updates
- `references/ui/images-and-media.md` — avatars, galleries, carousels, video, aspect ratios, placeholders
- `references/ui/cards-and-lists.md` — card variants, list views, infinite scroll, virtualization, skeletons
- `references/ui/microcopy-and-ux-writing.md` — button labels, error messages, empty states, confirmation copy
- `references/ui/scroll-patterns.md` — sticky elements, scroll-snap, infinite scroll, scrollbar styling
- `references/ui/design-tokens.md` — token naming, CSS custom properties, theme architecture, multi-brand
- `references/ui/atmosphere-and-texture.md` — gradient meshes, noise/grain, glassmorphism, geometric patterns, depth effects
- `references/ui/style-catalog.md` — 25 UI styles (glassmorphism, brutalism, aurora, etc.) with effects, best-for, quick-pick table
- `references/ui/product-type-guide.md` — 35 product types mapped to style, colors, fonts, and landing patterns
- `references/ui/palette-recipes.md` — 4 production-ready OKLCH palettes (SaaS, e-commerce, editorial, fintech), color-mix(), hue reference
- `references/ui/animation-libraries.md` — Framer Motion, GSAP, spring physics, easing library, performance rules

---

## Specialization: DOCS

Diátaxis-driven documentation. Every document serves exactly one reader need.

### The Diátaxis Compass

| | **Serves STUDY** | **Serves WORK** |
|---|---|---|
| **Practical steps** | **Tutorial** — lesson for newcomers, guaranteed success | **How-to** — recipe for competent users accomplishing a goal |
| **Theoretical knowledge** | **Explanation** — discussion, context, reasons | **Reference** — neutral dictionary of facts about the machinery |

**Cardinal sin**: mixing quadrants. When urge to mix → *link* to the other quadrant, don't merge.

### Detect Mode

| User says | Mode |
|---|---|
| "write docs/tutorial/README for X", "document this" | **WRITE** |
| "improve/rewrite/fix this doc", "this README is bad" | **IMPROVE** |
| "audit our docs", "restructure docs" | **AUDIT** |

### WRITE Mode

1. **Recon** — detect docs stack (Convention Detection above), read existing docs (tone, terminology,
   frontmatter schema, nav), read the actual code being documented (real API surface, real option
   names, real defaults — the code is the source of truth).
2. **Intake** — pin down: quadrant, target audience, reader's goal, scope. Answer from recon;
   ask only what the repo can't answer (one question at a time via `AskUserQuestion`).
3. **Outline gate** — file path(s), heading-level outline, quadrant per page, nav changes.
   **STOP and wait for approval.** Everything before this is cheap to change.
4. **Write** — follow per-quadrant playbook from `references/docs/`. Every snippet from the
   codebase; names copied exactly from source; defaults read from manifests, not recalled.
5. **Self-review** — score: Quadrant Purity, Audience Fit, Accuracy, Completeness, Followability,
   Voice, Stack Fitness. Fix anything under 4 before presenting.

### IMPROVE Mode

1. Classify the page's intended quadrant from location, title, content
2. Diff against the quadrant's standard — list concrete violations (stale claims, wrong audience, mixed purpose)
3. Verify every snippet, option name, version claim against current code
4. Rewrite preserving accurate, project-specific content; preserve the project's voice
5. Gate before splitting or moving pages (restructuring breaks links)

### AUDIT Mode

1. Inventory all docs pages (path + title)
2. Classify each by dominant quadrant; flag mixed, misfiled, duplicated, orphaned
3. Gap map — 4-quadrant grid for main user journeys; mark missing quadrants
4. Report: `page → current state → quadrant → action (keep/rewrite/split/merge/move/delete)`
5. Gate on approval before executing restructuring

### Style Non-Negotiables

- One idea per sentence. One purpose per paragraph. One quadrant per page.
- Address reader as "you" (tutorials: "we")
- Imperative mood: "Run the build", not "You can run the build"
- Ban: *simply, just, easy, obviously, of course*
- Present tense, active voice
- Every code block declares its language
- Headings are scannable claims: "Configure the webhook" beats "Configuration"
- Warnings come *before* the dangerous step, never after
- Never document behavior not verified in the code

### DOCS References (load on demand)

- `references/docs/tutorials.md` — tutorial playbook with templates
- `references/docs/how-to-guides.md` — how-to guide playbook
- `references/docs/reference.md` — reference/API docs playbook
- `references/docs/explanation.md` — concept/background page playbook
- `references/docs/developer-docs.md` — README, CONTRIBUTING, ARCHITECTURE, ADRs, changelogs
- `references/docs/style-and-voice.md` — full style guide, project voice calibration
- `references/docs/docs-stacks.md` — Fumadocs, Docusaurus, Starlight, MkDocs, VitePress, Mintlify

---

## Specialization: MAINTAIN

Repo-wide health work — operates on green main, not the working diff (that's ENGINEER's job
mid-feature). Five sub-modes share one engine and one triage discipline.

### The Shared Engine

Every sub-mode runs **DETECT → SCAN → TRIAGE → FIX → VERIFY**: detect the applicable tooling
(dep audit CLI, linter, test runner, dep-graph tool, package manager), scan for the full set of
findings, triage by severity × blast-radius into safe waves, fix one wave at a time with a
verification gate between waves, and re-verify the whole repo at the end. Never fix silently —
every wave is visible and gated. Full mechanics: `references/maintain/health-engine.md`.

### Sub-modes

| Mode | Detects | Fixes | Reference |
|---|---|---|---|
| **AUDIT** | Vulnerable dependencies (CVEs), risky code patterns (secrets, injection, missing authz) | Nothing auto-fixed — triages by severity × reachability, reports, gates on user for remediation path | `references/maintain/audit.md` |
| **DEBT** | Repo-wide lint/typecheck violations and suppressions (`@ts-ignore`, `# type: ignore`, `eslint-disable`) | One rule at a time, fixes the cause not the suppression | `references/maintain/debt.md` |
| **DEFLAKE** | Nondeterministic tests via repeated/shuffled/parallel runs | Root cause (shared state, timing, race, randomness) — never retry/skip/sleep to mask | `references/maintain/deflake.md` |
| **PRUNE** | Unused dependencies, unreferenced exports, orphaned files (tool evidence, not guesswork) | Reversible waves — devDeps first, runtime deps next, dynamic-ref-risk last; honors `preferences.maintain.protectedPaths` | `references/maintain/prune.md` |
| **UPGRADE** | Outdated dependencies vs. latest/wanted | Semver-gated waves — patches/minors batched, majors one at a time with changelog review; regenerates lockfile + re-tests per wave | `references/maintain/upgrade.md` |

### Detect Sub-mode

| User says | Sub-mode |
|---|---|
| "security audit", "are we vulnerable", "scan for CVEs" | **AUDIT** |
| "fix our lint warnings", "clear type errors", "get to strict mode" | **DEBT** |
| "fix flaky tests", "CI is flaky", "fails randomly" | **DEFLAKE** |
| "remove dead code", "what can we delete", "unused deps" | **PRUNE** |
| "upgrade our dependencies", "bump deps", "clear the Dependabot backlog" | **UPGRADE** |

If ambiguous (e.g. "clean up the repo"), ask which sub-mode(s) — don't guess and run all five.

### Non-Negotiables (all sub-modes)

- Never suppress, skip, or mask a finding to make a check pass — fix the cause.
- Every wave gated on user confirmation before the next, unless `preferences.autonomy == "auto-low-risk"` and the wave is low-risk.
- Re-run full verification (test/lint/typecheck/build) after every wave, not just at the end.
- Respect `preferences.maintain.protectedPaths` — never touch generated/vendored code.

---

## Gotchas (all specializations)

1. **Chaining without a gate** (ENGINEER) — never advance past a phase boundary without explicit "go"
2. **Asking what the codebase answers** — search configs, deps, test files before every question
3. **Parallel agents editing shared files** (ENGINEER) — assign ownership in DECOMPOSE; serialize when in doubt
4. **Rollback point mid-wave** (ENGINEER) — capture git hash before Wave 1 touches anything
5. **Board done without tests running** (ENGINEER) — mandatory tail tasks are the most-skipped; never mark
   completed without actual test/lint/build output on the board
6. **Generic fonts, gradients, emojis as icons** (UI) — #1 AI slop tells; use real icon libraries
7. **Mixing quadrants** (DOCS) — when you feel the urge to merge, link instead
8. **Documenting nonexistent features** (DOCS) — never write aspirational docs; stop and say so
9. **Suppressing findings to pass a check** (MAINTAIN) — `@ts-ignore`, retried flaky tests, or
   deleting a failing test are never fixes; find the cause. PRUNE especially: always honor
   `preferences.maintain.protectedPaths` — generated/vendored code looks unused to static analysis
10. **Auto-committing** (all) — suggest a commit message; never run `git commit`

---

## References

**Engineering specialization** — load entering ENGINEER mode:
- `references/engineering/intake-playbook.md` — question banks per type, design-tree traversal, calibration, anti-patterns
- `references/engineering/spec-writing.md` — template, section scaling, decision log, review rubric
- `references/engineering/approach-analysis.md` — when/how to propose multiple approaches, trade-off dimensions, project decomposition
- `references/engineering/board-format.md` — `.auto-engineer/board.md` spec, status transitions
- `references/engineering/dependency-graph-patterns.md` — DAG topology patterns, wave-assignment algorithm, ASCII rendering
- `references/engineering/execution-model.md` — wave orchestration, agent prompt template, scope-creep guard, mandatory tail tasks
- `references/engineering/verification-framework.md` — TDD per task, signals, evaluator integration
- `references/engineering/evaluator-protocol.md` — generator-evaluator separation, scored rubric, iterative refinement loop
- `references/engineering/migration-playbook.md` — call-site inventory, codemods, rollback, compat
- `references/engineering/context-reset.md` — when/how to reset context on long sessions, handoff document format
- `references/engineering/config-schema.md` — `.absolute.config.json` / `~/.absolute/config.json` full schema and precedence

**UI specialization** — load per task from `references/ui/` (see list above)

**Docs specialization** — load per quadrant from `references/docs/` (see list above)

**Maintain specialization** — load entering MAINTAIN mode:
- `references/maintain/health-engine.md` — shared DETECT→SCAN→TRIAGE→FIX→VERIFY mechanics
- `references/maintain/audit.md`, `debt.md`, `deflake.md`, `prune.md`, `upgrade.md` — one per sub-mode, load only the active one
