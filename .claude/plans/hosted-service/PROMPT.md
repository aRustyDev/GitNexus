# Review, ground and refine the GitNexus hosted-service backlog

> **Trigger prompt.** Paste everything below the horizontal rule into a fresh Claude Code
> session started in `~/repos/woven/forks/gitnexus`. Research and refinement only — it stops
> short of writing application code.
>
> Drafted 2026-08-19, alongside the Plane backlog it refines.

---

## Mission

A 23-item backlog exists in Plane for turning GitNexus into a multi-tenant internal service.
**It was written from a requirements brief, not from this codebase.** Your job is to take each
item to the source and turn intent into a grounded, sequenced plan.

For every item, answer four questions:

1. **Is it real?** Does the problem it describes actually exist in this code?
2. **Is it already done?** In whole or in part.
3. **What does it actually cost?** In call sites, files and subsystems — not adjectives.
4. **What does it depend on?** Both other items and facts not yet known.

**Deleting an item is a good outcome. So is "already built".** You are not here to validate the
backlog; you are here to find out what is true.

Stop at a refined plan. **Do not write application code, do not open PRs, do not push.**

## Scope boundary

This plan owns **application code in this repository only**.

- The **Helm chart** belongs to `infrastructure/helm-charts` →
  `.claude/plans/gitnexus-chart/`, tracked in Plane project `Infrastructure/IAC`. That plan is
  deliberately charting the **public upstream image with no code changes**, so it is not
  blocked on you — but it will accumulate dependencies on items here. When you change an
  item's shape, check whether the chart plan assumed otherwise.
- **Cluster plumbing** — ArgoCD, DNS legs, the `gitnxs` beads database, status-page entries —
  belongs to `infrastructure/infrastructure` → `.claude/plans/gitnexus-delivery/`.

## Authorization

- **Workflow orchestration is explicitly authorized.** One agent per item, or per cluster of
  related items, is a good shape here. Adversarially verify any finding that would delete or
  drastically resize an item before you act on it.
- Web research authorized: upstream repo and issues, LadybugDB, MCP specification, Neo4j /
  Qdrant / OpenSearch docs, PolyForm licensing.
- Subagents authorized.
- **Work in a git worktree on a feature branch**, never on `main`.

## The backlog

Plane workspace `forks`, project **GITNEXUS** (`2e835738-6bf8-4120-a0d5-a0178d1130ef`).
Credential `op://Work/plane-pat/token`, header `X-API-Key`, base `https://projects.woven`.
Items are `GITNEXUS-1` … `GITNEXUS-23`. **Fetch them rather than trusting any local copy** —
the descriptions carry the intent, and they may have been edited since.

Read the plan's `README.md` for the item table and the known-suspect list.

## Phase 0 — Licensing (BLOCKING, `GITNEXUS-1`)

GitNexus is **PolyForm Noncommercial 1.0.0** and a commercial "GitNexus Enterprise" tier exists.

Resolve, quoting operative clauses rather than paraphrasing: whether internal hosting at a
commercial organization is a permitted purpose; whether a dual or commercial license exists and
what it costs; whether maintaining this fork with internal patches and publishing derived
images internally stays inside the terms; and whether any dependency is independently
restrictive (LadybugDB, tree-sitter grammars, arctic-embed weights, ONNX runtime).

If it comes back **not permitted**, stop refining and instead produce the finding, the
commercial path, and a comparison of permissively-licensed alternatives. Do not quietly plan
around it.

Note **`GITNEXUS-17`** (GitHub App / blast-radius PR analysis) closely resembles the paid
Enterprise feature. Flag it explicitly in your verdict even if general hosting is fine.

## Phase 1 — Ground the codebase

Before judging any item, build the shared picture. Output to `RESEARCH.md`; every claim carries
a file path, a line, or a command you ran. Mark anything unproven **UNVERIFIED**.

Start from `ARCHITECTURE.md`, `RUNBOOK.md`, `MIGRATION.md`, `AGENTS.md`, `GUARDRAILS.md` and
`DoD.md` — then verify against the source, because docs drift.

- **Topology.** The monorepo packages (`gitnexus`, `gitnexus-web`, `gitnexus-shared`,
  `gitnexus-claude-plugin`, `gitnexus-cursor-integration`, `eval`, `pr-swarm-review`) and the
  CLI ↔ server ↔ web ↔ MCP boundaries.
- **The storage layer.** LadybugDB (`@ladybugdb/core`), schema in `lbug/schema.ts`, the
  per-repo `.gitnexus/lbug` layout, `repo-manager.ts`, and the global `registry.json`.
  **Count the call sites** — this number decides items 5, 6 and 7.
- **`lbug.lock`.** What it serializes: writers only or readers too, per repo or per process.
  Behaviour on unclean shutdown. **This is the single most consequential unknown in the
  backlog** and it gates `GITNEXUS-2`.
- **The pipeline.** Phase DAG, what is incremental today, runtime and memory on a large repo.
- **Search.** BM25 plus semantic vectors fused by RRF (K=60); the `Embedding` table; where each
  half is computed.
- **Embeddings.** arctic-embed-xs at 384 dimensions via `onnxruntime-node` /
  `@huggingface/transformers`, incremental by SHA1.
- **The MCP surface.** Tool inventory, transport (stdio / SSE / Streamable HTTP), whether there
  is an `Mcp-Session-Id` and whether server state is per-session.
- **The server.** `gitnexus serve`, Express 5, `cors`, `express-rate-limit`, `busboy`,
  `proxy-addr`. **Assume no authentication until proven otherwise.**
- **Existing deployment surface.** `Dockerfile.cli`, `Dockerfile.web`, `docker-compose.yaml`,
  `render.yaml`, `deploy/kubernetes/cluster-image-policy.yaml`, and how releases are cut today.

**An instance is already running on this machine** — ports 4173 (web) and 4747 (server), per
`local-dev/README.md` in the helm-charts repo. Interrogate it: enumerate the live MCP tools,
watch the transport and session headers on the wire, and test lock behaviour under concurrent
access. Read-only — do not reindex, mutate or restart anything you did not create.

## Phase 2 — Adjudicate each item

Output to `FINDINGS.md`, one entry per Plane item, in item order. Each entry carries:

- **Verdict** — `REAL` / `ALREADY DONE` / `PARTLY DONE` / `MIS-SCOPED` / `DELETE` / `BLOCKED`.
- **Evidence** — paths, call-site counts, observed behaviour.
- **True scope** — what the work actually is, once grounded.
- **Cost** — in files and subsystems touched. `"~40 call sites across
  src/core/{graph,search,pipeline}"` is an estimate; `"large refactor"` is not.
- **Dependencies** — on other items, and on unknowns still outstanding.
- **Recommendation** — keep as-is, resize, split, merge, defer, or delete. Say what would
  change your mind.

Give the known-suspect items real scrutiny: **#11** (embeddings already incremental), **#5**
(coupling depth unmeasured), **#2** (the lock decides everything downstream), **#13** and
**#18** (large permanent surfaces, both written as "potentially").

If a verdict would delete or drastically resize an item, **have an independent agent try to
refute it first.** A wrong deletion is far more expensive than a redundant item.

## Phase 3 — Sequence

Write `PLAN.md`. Order by **dependency, then by what de-risks the most for the least effort**.

Expect the true first tranche to be small: the licensing answer, the lock measurement, and
whatever `GITNEXUS-9` (multi-repo and git access) minimally needs to fetch a private repository.
Those three unblock most of the rest, and two are research rather than engineering.

Say explicitly which items the research proved unnecessary, and which turned out to be
prerequisites nobody had written down.

Where a decision is genuinely open — graph backend, isolation model, service seams — draft it
as an ADR in `.claude/plans/hosted-service/adrs/NNNN-<slug>.md`. Those are **open and
plan-scoped**. On acceptance, graduate to `docs/src/dev/adrs/` (creating that tree if needed) as
a move plus a rewrite into the present tense — never a copy. Once accepted, an ADR is immutable;
supersede, never edit. Graduate nothing the requester has not accepted.

## Phase 4 — Reflect the refinement back

Only after the requester has reviewed `FINDINGS.md` and `PLAN.md`.

**Plane (human-first)** — update the existing items rather than creating parallel ones: correct
descriptions where grounding changed the story, close anything found already done, and add
items for prerequisites the brief missed. Keep the register human: goals, intent, why the scope
changed. **Do not delete an item to record that it was unnecessary** — close it with the reason,
so the decision stays visible.

**beads (agent-first)** — database **`gitnxs`** on `bd.agents.woven:3306`. It **does not exist
yet** and this repo has no `.beads/`; creating it is owned by the `gitnexus-delivery` plan,
which must also add it to `local.backup_databases` in `shared/kube/beads/backup.tf` — a database
missing from that list has **no backup at all**. Coordinate; do not create it yourself as a side
effect.

Once it exists, derive beads from the Plane items with explicit linkage both ways. Beads are for
agents: token-efficient, explicitly instructed, hard acceptance criteria, every bead naming the
files it touches and the command that proves it done. Never run `bd dolt push`. Quote note text
with **single quotes** and use `--append-notes`, never `--notes "$(…)"`; assert every write
against `bd show <id> --json` — see `~/.claude/rules/tool-call-plumbing.md`.

## Deliverables

Under `.claude/plans/hosted-service/`, on a worktree feature branch. Keep `README.md`'s map and
status current.

| Document | Contents |
|---|---|
| `FINDINGS.md` | Licensing verdict first, then one adjudication per item, then the risk register. |
| `RESEARCH.md` | Phase 1 archaeology. Split to `research/nnnn-*.md` only if it outgrows one file. |
| `adrs/NNNN-*.md` | Open decisions. |
| `PLAN.md` | The sequenced backlog. |
| `QUESTIONS.md` | Open questions for the requester. |

Create each directory on its first real document; never scaffold. Numbering is global per kind,
so gaps are expected — note them in `README.md` rather than renumbering.

## Non-goals

No application code written. No PR opened, nothing pushed. No Plane item edited or bead created
before the requester reviews the findings. No chart work — that is `gitnexus-chart`. No cluster
touched, no beads database created — that is `gitnexus-delivery`. No ADR graduated to
`docs/src/`.

## Working rules

- **The backlog is a hypothesis, not a specification.** It was written without reading this
  code. Contradicting it with evidence is the job.
- Report faithfully. "This item is already implemented" and "this cannot be done under the
  license" are both more valuable than a plan that hides them.
- Every estimate gets a basis.
- **Measure the lock before designing anything around it.** Almost everything downstream —
  whether an external graph store is needed at all — turns on that one answer.
