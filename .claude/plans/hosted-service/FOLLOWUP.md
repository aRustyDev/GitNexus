# Follow-up: act on the grounded backlog

> **Second prompt.** Fire this **after** `PROMPT.md`'s session has landed `RESEARCH.md`,
> `FINDINGS.md` (one adjudication per Plane item) and `PLAN.md`, and **after a human has
> reviewed them**. Paste everything below the horizontal rule into a fresh Claude Code session
> started in `~/repos/woven/forks/gitnexus`.
>
> Drafted 2026-08-19 alongside `PROMPT.md`. If the grounding contradicted anything here,
> **the grounding wins**.

---

## Mission

Two jobs, in order:

1. **Reflect the refinement back into Plane** — the backlog currently describes a codebase
   nobody had read. Make it describe the real one.
2. **Execute the first tranche** from `PLAN.md` — and only the first tranche.

## Preconditions — check, do not assume

Stop and report if any fails:

1. **`FINDINGS.md` carries a verdict for every item `GITNEXUS-1` … `GITNEXUS-23`**, and a human
   has reviewed it. Partial grounding is not a mandate to start editing the backlog.
2. **`GITNEXUS-1` (licensing) is resolved.** If it came back *not permitted*, **stop entirely**
   — do not write code, do not update the backlog beyond recording the verdict. Report and wait.
3. **The `gitnxs` beads database exists.** It is provisioned by
   `infrastructure/infrastructure` → `.claude/plans/gitnexus-delivery/`, and must appear in
   `local.backup_databases` in `shared/kube/beads/backup.tf` or it has no backup at all. If it
   does not exist, **do not create it here** — proceed with Plane only and say beads is blocked.

## Job 1 — reflect the refinement into Plane

Workspace `forks`, project **GITNEXUS** (`2e835738-6bf8-4120-a0d5-a0178d1130ef`).

**Use the existing `plane` skill rather than hand-rolling API calls.** It lives on branch
`feat/plane-api-skill` in the infrastructure repo at `.claude/skills/plane/` and carries a
workspace/slug table, an endpoint reference, and `scripts/plane-api.sh`. A prior session
brute-force probed for the workspace slug because it did not know this existed — do not repeat
that. Credential is `op://Work/plane-pat/token`, header `X-API-Key`, base `https://projects.woven`.

Then, per verdict:

| Verdict | Action |
|---|---|
| `REAL`, scope unchanged | Leave it. Do not churn descriptions for style. |
| `REAL`, resized | Update the description to say what it actually is, and **why the scope changed**. |
| `PARTLY DONE` | Update to describe only the remainder; note what already exists and where. |
| `ALREADY DONE` | **Close with the reason and the evidence. Do not delete.** |
| `MIS-SCOPED` | Split or merge. Link parent/child explicitly in both directions. |
| `DELETE` | Close, with the refutation recorded. A closed item with a reason is documentation; a deleted one is amnesia. |
| `BLOCKED` | Say what on, and link it. |

**Add items for prerequisites the brief missed** — grounding almost always surfaces some.

Keep the human register: goals, intent, why. Never let this become a changelog of scope edits.

**Assert every write.** Read the item back and confirm the text survived; a success code proves
the call was accepted, never that the content is intact.

## Job 2 — derive beads

Database **`gitnxs`** on `bd.agents.woven:3306`, once it exists.

- Model `.beads/config.yaml` and `.beads/metadata.json` on the helm-charts repo's, changing only
  the database name. `bd` is a pure client — `dolt.auto-start: false`.
- **Never run `bd dolt push`.** Denied by design.
- Beads are the agent register: token-efficient, explicitly instructed, hard acceptance
  criteria. Every bead names **the files it touches** and **the command that proves it done**,
  and carries its dependencies. No motivational prose — that lives in Plane.
- Link each bead to its Plane item and back.
- **Quote note text with single quotes.** `$(…)` *and backticks inside double quotes* both
  execute — a measured incident silently deleted words from a note while reporting success. Use
  `--append-notes`, never `--notes`.
- **Assert against `bd show <id> --json`, never the rendered view** — the renderer word-wraps
  and breaks inside hyphenated words, producing false negatives that invite a corrective write
  against undamaged data.

## Job 3 — execute the first tranche only

Take the top tranche of `PLAN.md` and nothing below it. Expect it to be small — likely the lock
measurement, whatever `GITNEXUS-9` minimally needs to fetch a private repo, and little else.

- **One branch per work item.** `feat/<slug>`, worktree per unit of work.
- This repo has real gates: `npm run lint`, `npm run format:check`, and the guardrails in
  `GUARDRAILS.md` / `DoD.md`. **Read `DoD.md` before claiming anything done.** Note
  `.prettierignore` excludes `*.md`, so docs commits do not need formatting.
- Tests exist (`TESTING.md`). A change to the pipeline or storage layer without a test is not
  done.
- **Upstream is alive** — `aRustyDev/GitNexus` tracks `abhigyanpatwari/GitNexus`. Keep patches
  rebasable and narrow. Anything generally useful is a candidate to contribute upstream, which
  is also the cheapest way to avoid carrying it forever.

## Stop conditions

Stop and report rather than pressing on if: the licensing verdict is negative; a first-tranche
item turns out to be far larger than `FINDINGS.md` estimated (the estimate was wrong — say so
rather than absorbing it); or the work would require changing the chart or the cluster, which
belong to the other two plans.

## Ground rules

- Worktree, feature branch, Conventional Commits. Never `main`.
- Do not push, open a PR, or merge without being asked.
- Report faithfully. "The estimate was wrong" and "this is already implemented" are findings,
  not failures.
