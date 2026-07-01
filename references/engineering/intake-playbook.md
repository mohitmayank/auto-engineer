<!-- Part of the Auto Engineer skill. Load this file when the agent needs detailed guidance on conducting the INTAKE phase, including question banks, scaling rules, and example sessions. -->

# Intake Playbook

The intake phase is the foundation of every Auto Engineer execution. A thorough intake prevents rework, missed requirements, and scope creep. This playbook provides the full question bank, scaling rules, and example sessions.

---

## Design Tree Traversal

Every unit of work is a tree of decisions. Walk it **depth-first**, resolving each branch
completely before moving to siblings. This prevents half-explored requirements from haunting
the implementation later.

### Rules
1. **Root first** — start with purpose. Why does this exist? What problem does it solve?
2. **Depth before breadth** — explore the first child fully before siblings.
3. **Resolve before advancing** — a node is resolved when you have a clear answer, a concrete decision, or an explicit deferral. Never leave a node ambiguous.
4. **Backtrack on dead ends** — if a branch leads to "we don't need this," mark it explicitly out of scope and backtrack.
5. **Dependency edges** — if a node depends on another branch, resolve the blocker first.

### Example tree: "Add a commenting system"
```
commenting-system
├── purpose (who comments? what is commentable?)
├── data-model (schema, threading flat vs nested, storage)
├── permissions (create/edit/delete, moderation)
├── ui (input, list, threading display, empty state)
├── real-time (needed? transport? optimistic updates?)
├── notifications (notify on reply? channel?)
└── edge-cases (deleted parent, deleted post, concurrent edits, spam)
```
By the time you reach `notifications`, the threading decision is already resolved, so you know
whether replies even exist. Upstream decisions constrain downstream ones — always interview in
tree order: purpose → data model → behavior → UI → edge cases.

---

## Question Bank by Task Type

### Feature Development
| # | Question | Purpose |
|---|----------|---------|
| 1 | What feature needs to be built? Describe the user-facing behavior. | Problem statement |
| 2 | What does "done" look like? List specific acceptance criteria. | Success criteria |
| 3 | Are there existing patterns or conventions in the codebase we should follow? | Constraints |
| 4 | Which files, modules, or components will this touch? | Scope mapping |
| 5 | Does this depend on any external APIs, services, or libraries? | Dependencies |
| 6 | What are the known edge cases or error scenarios? | Edge cases |
| 7 | How should this be tested? Unit, integration, e2e? | Testing strategy |
| 8 | Does this need documentation updates? API docs, README, changelog? | Documentation |
| 9 | Any backwards compatibility or migration concerns? | Rollout |
| 10 | Which parts are highest priority if we need to split delivery? | Priority |

### Bug Fix
| # | Question | Purpose |
|---|----------|---------|
| 1 | What is the bug? Describe the expected vs actual behavior. | Problem statement |
| 2 | How do we reproduce it? Steps, environment, conditions. | Reproduction |
| 3 | What is the impact? Who is affected and how severely? | Priority |
| 4 | When did this start happening? Any recent changes? | Root cause hints |
| 5 | Are there related bugs or known issues? | Context |
| 6 | What is the fix criteria? When is this bug "fixed"? | Success criteria |

### Refactor
| # | Question | Purpose |
|---|----------|---------|
| 1 | What code needs refactoring and why? | Problem statement |
| 2 | What is the desired end state? | Success criteria |
| 3 | Must the refactor be backwards-compatible? | Constraints |
| 4 | What is the test coverage of the code being refactored? | Safety net |
| 5 | Can this be done incrementally or must it be all-at-once? | Strategy |
| 6 | Are there downstream consumers that will be affected? | Impact |

### Greenfield Project
| # | Question | Purpose |
|---|----------|---------|
| 1 | What is the project and what problem does it solve? | Problem statement |
| 2 | Who is the target user? | Context |
| 3 | What are the core features for v1? | Scope |
| 4 | What tech stack and conventions should we use? | Constraints |
| 5 | Are there reference implementations or designs to follow? | Patterns |
| 6 | What external services or APIs will we integrate with? | Dependencies |
| 7 | What is the testing strategy? | Testing |
| 8 | What does the deployment/infra look like? | Infrastructure |
| 9 | What documentation is needed? | Documentation |
| 10 | What is the priority order of features? | Priority |

### Planning / Breakdown
Use when the user wants a vague goal turned into a sequenced plan rather than an immediate build.
| # | Question | Purpose |
|---|----------|---------|
| 1 | What is the end goal, stated as an outcome? | North star |
| 2 | What are the milestones between here and there? | Sequencing |
| 3 | What must ship first to unblock the rest? | Critical path |
| 4 | What is already done or in flight? | Current state |
| 5 | What are the hard deadlines or constraints? | Boundaries |
| 6 | What can be deferred to a later phase? | Scope control |

### Migration
| # | Question | Purpose |
|---|----------|---------|
| 1 | What is being migrated and to what? (e.g., v2 to v3, JS to TS) | Problem statement |
| 2 | What is the scope? Full migration or incremental? | Strategy |
| 3 | Must the old and new coexist during migration? | Constraints |
| 4 | What is the rollback plan if something goes wrong? | Safety |
| 5 | Are there breaking changes to account for? | Risk |
| 6 | What is the test coverage of the code being migrated? | Safety net |
| 7 | What is the priority order of modules to migrate? | Priority |

---

## Scaling Rules

### When to Ask 3 Questions (Simple)
- Task touches 1-2 files
- Clear, well-defined scope ("add a button that does X")
- No external dependencies
- Existing patterns to follow
- **Always ask**: Problem statement, Success criteria, Constraints

### When to Ask 5 Questions (Medium)
- Task touches 3-5 files or 2+ components
- Some ambiguity in requirements
- May involve external APIs
- **Add**: Existing code context, Dependencies

### When to Ask 8-10 Questions (Complex)
- Task touches 5+ files or is cross-cutting
- Greenfield project or major refactor
- External services, migrations, or rollout concerns
- **Add**: Edge cases, Testing strategy, Documentation, Rollout, Priority

### Complexity Detection Heuristic
Ask yourself:
1. How many files/components will this touch? (1-2: simple, 3-5: medium, 5+: complex)
2. Are there external dependencies? (no: simpler, yes: more complex)
3. Is the scope well-defined? (yes: simpler, no: more complex)
4. Does this involve data migration or backwards compatibility? (yes: complex)

---

## Question Calibration

### Multiple Choice vs Open-Ended

| Use Multiple Choice When | Use Open-Ended When |
|---|---|
| There are 2-4 well-known options | The answer space is unbounded |
| You want to anchor the user on realistic choices | You want to discover requirements you haven't thought of |
| The user is non-technical and might not know terminology | The user is the domain expert and you need their mental model |
| Speed matters - reduce back-and-forth | Depth matters - you need their full reasoning |

**Examples:**

- **Multiple choice**: "For threading, should comments be: (a) flat - single level, (b) nested - up to 3 levels, (c) nested - unlimited depth?"
- **Open-ended**: "Describe how you envision the moderation workflow when a comment is flagged."

### When a Question Is Too Broad

A question is too broad if the user would need more than 3 sentences to answer it well. Split it.

| Too Broad | Better |
|---|---|
| "How should the notification system work?" | "Should notifications be in-app only, email only, or both?" followed by "For in-app notifications, should they appear as a badge, a dropdown, or a full page?" |
| "What are the security requirements?" | "Who should be able to access this resource?" followed by "Do we need rate limiting on this endpoint?" |
| "How should errors be handled?" | "When the API returns a 4xx error, what should the user see?" followed by "Should we retry on network failures? How many times?" |

### When to Merge vs Split Questions

**Merge** when two questions are so tightly coupled that answering one without the other is meaningless:
- "Should comments support threading, and if so, what is the max nesting depth?" (threading yes/no and depth are inseparable)

**Split** when a question bundles independent decisions:
- Bad: "Should we use WebSockets for real-time updates and Redis for caching?"
- Good: Ask about real-time transport separately from caching strategy.

### Calibration Rules

1. **One decision per question.** If your question contains "and," consider splitting.
2. **No compound conditionals.** Instead of "If X, should we do Y or Z, and if not X, should we do A?" - first resolve X, then ask the follow-up.
3. **Ground in the codebase.** Reference what you found: "I see you're using Express with middleware-based routing. Should the new auth endpoints follow the same pattern?"
4. **Offer a recommendation when you can.** "I'd suggest cursor-based pagination here because your dataset will grow over time. Does that work, or do you need offset-based for a specific reason?"
5. **Timebox complexity.** If a question opens a 20-minute rabbit hole, flag it: "This is a significant decision. Should we resolve it now or defer and use a simple placeholder?"

---

## Extracting Implicit Requirements

Users often omit critical requirements. Watch for these patterns:

| User Says | Implicit Requirement |
|---|---|
| "Add a login page" | Auth system, session management, error handling, redirect logic |
| "Make it faster" | Needs benchmarks before and after, specific performance targets |
| "Support dark mode" | Theme system, all components must be theme-aware, persistence |
| "Add search" | Indexing strategy, debounce, empty state, pagination |
| "Deploy to production" | CI/CD, environment config, monitoring, rollback plan |

When you detect implicit requirements, surface them as follow-up questions rather than assuming.

---

## Example Intake Session

### Task: "Add user authentication to our Next.js app"

**Detected complexity**: Complex (cross-cutting, external dependencies, security-sensitive)

**Questions asked**:
1. **Problem**: "What authentication method do you need? Email/password, OAuth providers (Google, GitHub), magic links, or a combination?"
2. **Success criteria**: "What does 'authenticated' look like? Protected routes, user profile page, session persistence across browser restarts?"
3. **Constraints**: "Are there existing auth patterns in the codebase, or is this the first auth implementation? Any preferred libraries (NextAuth, Clerk, custom)?"
4. **Existing code**: "Is there already a user model or database schema? Any existing API routes we should integrate with?"
5. **Dependencies**: "Which OAuth providers need to be supported? Do we need a database for sessions/users?"
6. **Edge cases**: "How should we handle: expired sessions, multiple tabs, password reset, account lockout?"
7. **Testing**: "What auth flows need e2e tests? Is there an existing test setup?"
8. **Docs**: "Do we need API documentation for auth endpoints?"
9. **Rollout**: "Is this for new users only, or do existing users need migration?"
10. **Board persistence**: "Should the `.auto-engineer/` board be git-tracked or gitignored?"

**Intake Summary** (written to board):
```
## Intake Summary
- Task: Add email/password + Google OAuth authentication to Next.js app
- Library: NextAuth.js v5
- Database: Existing Prisma + PostgreSQL setup
- Protected routes: /dashboard, /settings, /api/*
- Success: User can register, login, logout, and access protected routes
- Edge cases: Session expiry shows login prompt, password reset via email
- Testing: E2e tests for login, register, OAuth flow, protected route redirect
- Board: gitignored (local working state)
```

---

## Anti-Patterns

These are the most common mistakes during a design interview. Each one wastes the user's time or produces an incomplete design.

### 1. Asking Questions the Codebase Can Answer

**Wrong**: "What database are you using?"
**Right**: Search `package.json` and config files first. Then say: "I see you're using Prisma with PostgreSQL."

**Why it matters**: It signals you haven't done your homework. The user will lose confidence that you're thorough enough to build the feature correctly.

### 2. Batching Multiple Unrelated Questions

**Wrong**: "What should the notification bell look like, and how should we handle real-time transport, and do you need email notifications?"
**Right**: Ask one question at a time, depth-first. Resolve delivery transport before asking about email. Resolve the bell UI separately.

**Why it matters**: The user will answer the easiest question and skip the hard ones. You will think you have answers when you have gaps.

### 3. Asking About Implementation Before Purpose

**Wrong**: "Should we use WebSockets or SSE?"
**Right**: First ask what the feature does and how frequently events occur. Then recommend the transport based on the requirements.

**Why it matters**: Implementation decisions without context are coin flips. Purpose determines constraints, constraints determine implementation.

### 4. Accepting Vague Answers Without Follow-Up

**Wrong**:
- User: "It should handle errors gracefully."
- Agent: "Got it." (moves on)

**Right**:
- User: "It should handle errors gracefully."
- Agent: "Can you give me an example? When a notification fails to deliver, should we: (a) silently retry, (b) show an error toast to the user, or (c) log it and move on?"

**Why it matters**: "Gracefully" means something different to every person. Vague requirements become bugs.

### 5. Not Exploring Error and Edge Case Branches

**Wrong**: Only asking about the happy path - what happens when everything works.
**Right**: For every feature branch, ask: "What happens when this fails? What happens at the boundaries (0 items, 10,000 items, very long input)?"

**Why it matters**: Edge cases are where most bugs live. If you don't ask, the user won't volunteer them, and you will discover them during implementation when they are 10x more expensive to resolve.

### 6. Asking Leading Questions

**Wrong**: "We should use Redis for caching here, right?"
**Right**: "Do you need caching for this endpoint? If so, what is the acceptable staleness - a few seconds, minutes, or hours?"

**Why it matters**: Leading questions confirm your bias instead of discovering the actual requirement. The user will often agree with your suggestion even if it is wrong.

### 7. Skipping the "Out of Scope" Conversation

**Wrong**: Assuming everything mentioned is in scope for v1.
**Right**: After mapping the full design tree, explicitly ask: "Which of these branches are v1 vs later?"

**Why it matters**: Without explicit scoping, the feature grows silently. You will build things the user did not need yet, and miss things they needed now.

### 8. Interviewing in the Wrong Order

**Wrong**: Asking about UI styling before understanding the data model.
**Right**: Follow the tree: purpose - data model - behavior - UI - edge cases.

**Why it matters**: Upstream decisions constrain downstream ones. If you design the UI before the data model, you may design something the data cannot support.
