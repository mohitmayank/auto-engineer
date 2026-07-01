<!-- Part of the Auto Engineer skill. Load this file when reading, writing, or resolving `.absolute.config.json` / `~/.absolute/config.json`, or running first-run setup. -->

# Config Schema

Auto Engineer caches detected conventions and user preferences in two optional JSON files
instead of re-detecting the stack on every invocation. Both files share the same
`conventions` + `preferences` shape; the global file wraps them under `defaults`/`projects`.

---

## `.absolute.config.json` (project, committed)

```json
{
  "version": 1,
  "conventions": {
    "packageManager": "npm",
    "languages": ["typescript"],
    "test": "npm test",
    "lint": "npm run lint",
    "typecheck": "npm run typecheck",
    "format": "npm run format",
    "build": "npm run build",
    "ci": ".github/workflows/ci.yml",
    "defaultBranch": "main",
    "ui": {
      "framework": "react",
      "styling": "tailwind",
      "iconLibrary": "lucide-react",
      "componentLib": "shadcn",
      "tokensPath": "src/styles/tokens.css"
    },
    "docs": {
      "stack": "fumadocs",
      "dir": "docs"
    }
  },
  "preferences": {
    "outputStyle": "normal",
    "autonomy": "gate-all",
    "tdd": "strict",
    "specDir": "docs/plans",
    "boardTracking": "gitignored",
    "families": ["build", "ui", "docs", "maintain"],
    "maintain": {
      "protectedPaths": ["dist/**", "vendor/**", "**/*.generated.*"],
      "deflakeRuns": 20
    }
  }
}
```

`conventions.ui` and `conventions.docs` are omitted entirely when nothing is detected (no UI
deps, no docs stack) — keep the file minimal.

## `~/.absolute/config.json` (user/global, machine-local — never committed)

```json
{
  "version": 1,
  "defaults": {
    "preferences": {
      "outputStyle": "normal",
      "autonomy": "gate-all",
      "tdd": "strict",
      "specDir": "docs/plans",
      "families": ["build", "ui", "docs", "maintain"]
    }
  },
  "projects": {
    "/Users/you/dev/your-repo": {
      "conventions": { "packageManager": "pnpm", "test": "pnpm test" },
      "preferences": { "outputStyle": "terse" }
    }
  }
}
```

## Field Meanings

| Field | Values | Effect |
|---|---|---|
| `conventions.*` | strings | Cached stack/scripts every phase runs through instead of re-detecting. |
| `conventions.format` | string | Formatter script — MAINTAIN sub-modes run it instead of re-detecting. |
| `conventions.ui` | object | Cached `framework`/`styling`/`iconLibrary`/`componentLib`/`tokensPath` — UI specialization adopts these instead of assuming Tailwind or guessing the icon library. |
| `conventions.docs` | object | Cached docs `stack` + `dir` — DOCS specialization skips marker-file re-detection. |
| `preferences.outputStyle` | `normal` \| `terse` | Verbosity of responses. |
| `preferences.autonomy` | `gate-all` \| `auto-low-risk` | Whether MAINTAIN sub-modes auto-apply safe waves without a gate. |
| `preferences.tdd` | `strict` \| `pragmatic` | ENGINEER's test-first rigor. |
| `preferences.specDir` | path | Where SPEC / SPEC-lite write design docs. |
| `preferences.boardTracking` | `gitignored` \| `git-tracked` | Whether `.auto-engineer/board.md` is committed — set once instead of asked every INTAKE. |
| `preferences.families` | subset of `["build","ui","docs","maintain"]` | Trims which specializations are offered when the prompt is ambiguous. |
| `preferences.maintain.protectedPaths` | glob[] | Never-delete globs the PRUNE sub-mode honors (generated/vendored code). |
| `preferences.maintain.deflakeRuns` | int | Default N repeat-runs the DEFLAKE sub-mode uses to establish flakiness. |

---

## Precedence

At the start of any specialization, resolve effective config by overlaying, highest wins:

1. `./.absolute.config.json` (project, committed)
2. `~/.absolute/config.json` → `projects["<cwd absolute path>"]`
3. `~/.absolute/config.json` → `defaults`

Shallow-merge `conventions` and `preferences` separately; merge the nested `conventions.ui`,
`conventions.docs`, and `preferences.maintain` blocks one level deeper so a partial override
doesn't drop sibling keys. If neither file exists, there is no config — soft-recommend
first-run setup (see SKILL.md "Config Reading") and fall back to on-the-fly detection.

## Gotchas

1. **Overwriting a hand-tuned config.** Always resolve first; merge, don't clobber.
2. **Caching wrong commands.** A cached `test`/`lint` script that doesn't match CI poisons
   every later invocation — verify each detected script actually runs before trusting it.
3. **Committing the global file.** `~/.absolute/config.json` is machine-local; only
   `.absolute.config.json` is committed.
4. **Stale conventions.** A renamed branch or swapped package manager silently misroutes
   commands — re-run first-run setup after stack changes.
5. **Treating config as a gate.** It isn't. Every specialization proceeds without it; config
   is an optimization, not a prerequisite.
