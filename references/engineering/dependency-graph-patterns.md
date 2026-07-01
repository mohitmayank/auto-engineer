<!-- Part of the Auto Engineer skill. Load this file when the agent needs guidance on decomposing tasks into dependency graphs, common DAG patterns, ASCII rendering, and wave assignment. -->

# Dependency Graph Patterns

This reference covers how to decompose tasks into dependency graphs, common patterns, the ASCII rendering format, and the wave assignment algorithm.

---

## Identifying Dependencies

A task B depends on task A if:
- B needs code/files that A creates
- B extends or modifies A's output
- B tests functionality that A implements
- B documents behavior that A defines
- B configures infrastructure that A requires

A task B does NOT depend on A if:
- They modify different files with no shared interfaces
- They implement independent features
- They can be tested in isolation

### Dependency Checklist
For each pair of tasks, ask:
1. Does task B need any file that task A creates? -> dependency
2. Does task B import or use any function/type that task A defines? -> dependency
3. Does task B test code that task A writes? -> dependency
4. Can task B's tests pass without task A being complete? -> if yes, no dependency

---

## Common DAG Patterns

### Linear Chain
One task after another. No parallelism possible.
```
AE-001 --> AE-002 --> AE-003 --> AE-004
```
**When it occurs**: Sequential migrations, step-by-step setup
**Waves**: Each task is its own wave (worst case for parallelism)

### Fan-Out
One task fans out to many independent tasks.
```
            +---> AE-002
            |
AE-001 ----+---> AE-003
            |
            +---> AE-004
```
**When it occurs**: After initial setup, multiple independent features branch off
**Waves**: Wave 1 = AE-001, Wave 2 = AE-002 + AE-003 + AE-004

### Fan-In
Many independent tasks converge to one.
```
AE-001 ---+
           |
AE-002 ---+---> AE-004
           |
AE-003 ---+
```
**When it occurs**: Integration testing after parallel implementation
**Waves**: Wave 1 = AE-001 + AE-002 + AE-003, Wave 2 = AE-004

### Diamond
Fan-out followed by fan-in.
```
            +---> AE-002 ---+
            |                |
AE-001 ----+                +---> AE-005
            |                |
            +---> AE-003 ---+
            |                |
            +---> AE-004 ---+
```
**When it occurs**: Setup -> parallel features -> integration
**Waves**: Wave 1 = AE-001, Wave 2 = AE-002 + AE-003 + AE-004, Wave 3 = AE-005

### Independent Clusters
Multiple disconnected sub-graphs that can run entirely in parallel.
```
Cluster A:  AE-001 --> AE-002
Cluster B:  AE-003 --> AE-004 --> AE-005
Cluster C:  AE-006
```
**When it occurs**: Unrelated features being built simultaneously
**Waves**: Wave 1 = AE-001 + AE-003 + AE-006, Wave 2 = AE-002 + AE-004, Wave 3 = AE-005

### Layered Architecture
Tasks organized by architectural layers.
```
Layer 1 (infra):    AE-001, AE-002
Layer 2 (data):     AE-003, AE-004    (depend on Layer 1)
Layer 3 (logic):    AE-005, AE-006    (depend on Layer 2)
Layer 4 (UI):       AE-007, AE-008    (depend on Layer 3)
Layer 5 (tests):    AE-009, AE-010    (depend on Layer 4)
```
**When it occurs**: Full-stack feature development
**Waves**: One wave per layer

---

## Wave Assignment Algorithm

### Algorithm (Topological Sort + Depth Grouping)

```
function assignWaves(tasks):
  // Calculate depth for each task
  for each task in tasks:
    if task has no dependencies:
      task.depth = 0
    else:
      task.depth = max(dependency.depth for dependency in task.dependencies) + 1

  // Group by depth
  waves = group tasks by task.depth

  // Wave 1 = depth 0, Wave 2 = depth 1, etc.
  return waves
```

### Rules
1. Tasks with no dependencies are always Wave 1
2. A task's wave = max(wave of its dependencies) + 1
3. All tasks in the same wave can execute in parallel
4. Waves execute in strict sequential order
5. If a wave has only 1 task, it still counts as a wave (serial execution)

### Optimization
- If two tasks are in the same wave but modify the same file, move one to a later wave to prevent conflicts
- If a wave has more tasks than available parallel agents, split it into sub-waves

---

## ASCII Graph Rendering Format

### Conventions
- `-->` for dependency edges (A --> B means "B depends on A")
- `+` for branch points
- `|` for vertical connections
- Indent child tasks under their parents
- Include task type and title in brackets

### Standard Format
```
Task Graph:
  AE-001 [type: title]
    |
    +---> AE-002 [type: title]
    |       |
    |       +---> AE-004 [type: title]
    |
    +---> AE-003 [type: title]
```

### With Wave Annotations
```
Task Graph:
  [W1] AE-001 [config: Init project structure]
         |
         +---> [W2] AE-002 [code: Database schema]
         |            |
         |            +---> [W3] AE-004 [code: User model]
         |
         +---> [W2] AE-003 [code: API router setup]

Wave Summary:
  Wave 1 (1 task):  AE-001
  Wave 2 (2 tasks): AE-002, AE-003  [parallel]
  Wave 3 (1 task):  AE-004
```

---

## Example: Full-Stack Feature Decomposition

### Task: "Add a commenting system to blog posts"

**Sub-tasks identified:**
- AE-001: Create Comment database model and migration (config, S)
- AE-002: Create Comment API endpoints - CRUD (code, M)
- AE-003: Create CommentList UI component (code, M)
- AE-004: Create CommentForm UI component (code, S)
- AE-005: Wire API to UI with data fetching (code, M)
- AE-006: Add comment notification system (code, M)
- AE-007: Write API integration tests (test, M)
- AE-008: Write UI component tests (test, S)
- AE-009: Update API documentation (docs, S)

**Dependencies:**
- AE-002 depends on AE-001 (needs the model)
- AE-003 depends on nothing (can use mock data)
- AE-004 depends on nothing (can use mock data)
- AE-005 depends on AE-002, AE-003, AE-004 (wires them together)
- AE-006 depends on AE-002 (needs API to trigger notifications)
- AE-007 depends on AE-002 (tests the API)
- AE-008 depends on AE-003, AE-004 (tests the components)
- AE-009 depends on AE-002 (documents the API)

**Graph:**
```
  AE-001 [config: Comment model + migration]
    |
    +---> AE-002 [code: Comment CRUD API]
    |       |
    |       +---> AE-005 [code: Wire API to UI] (also depends on AE-003, AE-004)
    |       +---> AE-006 [code: Notification system]
    |       +---> AE-007 [test: API integration tests]
    |       +---> AE-009 [docs: API documentation]
    |
    (independent)
    AE-003 [code: CommentList component]
    |       |
    |       +---> AE-005 (see above)
    |       +---> AE-008 [test: UI component tests] (also depends on AE-004)
    |
    AE-004 [code: CommentForm component]
```

**Wave Assignment:**
```
  Wave 1 (3 tasks): AE-001, AE-003, AE-004    [parallel]
  Wave 2 (1 task):  AE-002                      [serial - needs AE-001]
  Wave 3 (4 tasks): AE-005, AE-006, AE-007, AE-008, AE-009  [parallel]
```

Note: AE-005 depends on AE-002 (Wave 2) AND AE-003, AE-004 (Wave 1), so it goes in Wave 3. AE-008 depends on AE-003, AE-004 (Wave 1), so it could be Wave 2, but since it also tests the components, waiting for Wave 2 to complete is cleaner.
