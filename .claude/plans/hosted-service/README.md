# hosted-service

Turning GitNexus from a single-user local tool into a **multi-tenant internal service** —
queried by a fleet of agents over MCP, browsable by humans over the web UI.

**This plan owns application code only.** The Helm chart lives in
`infrastructure/helm-charts` → `.claude/plans/gitnexus-chart/`. Cluster plumbing (ArgoCD, DNS,
beads provisioning) lives in `infrastructure/infrastructure` →
`.claude/plans/gitnexus-delivery/`.

**Status (2026-08-19): backlog seeded, not yet grounded.** 23 high-level work items exist in
Plane. None has been checked against the real codebase.

## What this plan is for

The Plane backlog was written **from a requirements brief, not from the code**. It captures
intent accurately and technical reality only approximately. Several items are probably
mis-scoped, at least one is probably already partly built, and one may be forbidden outright.

So this plan is not "design the work" — the work is already sketched. It is **review, ground,
and refine**: take each item to the source, find out what is actually true, and turn a wish
list into a sequenced backlog with honest estimates.

Deleting an item is a good outcome. So is discovering it is already done.

## The backlog

Plane workspace `forks`, project **GITNEXUS** (`2e835738-6bf8-4120-a0d5-a0178d1130ef`).

| # | Item | Label | Priority |
|---|---|---|---|
| 1 | Resolve GitNexus licensing for internal hosting | foundation | **urgent** |
| 2 | Establish a multi-tenant service baseline | foundation | high |
| 3 | Add authentication and per-agent identity to the server | foundation | high |
| 4 | Support a remote embeddings provider | foundation, pluggable-backend | high |
| 5 | Configurable graph backend (external openCypher) | pluggable-backend | medium |
| 6 | Configurable vector store (Qdrant) | pluggable-backend | medium |
| 7 | Configurable search index (OpenSearch/Elasticsearch) | pluggable-backend | medium |
| 8 | Configurable cache (Valkey/Redis) | pluggable-backend | low |
| 9 | Configurable git server and repo targeting, with multi-repo support | foundation | high |
| 10 | Auto-reindexing and the auto-updating code wiki | platform | medium |
| 11 | Incremental indexing | platform | medium |
| 12 | WebSocket channels for live repository change notifications | integration | medium |
| 13 | Split into independently scalable services | platform | low |
| 14 | Human web UI as a deployable frontend | platform | low |
| 15 | Configurable custom LSP for additional language coverage | platform | low |
| 16 | Additional language support: IaC and emerging languages | roadmap | low |
| 17 | GitHub App for PR automation and blast-radius analysis | integration | low |
| 18 | Evaluate a GitNexus Kubernetes Operator | platform, roadmap | low |
| 19 | Org- and repo-agnostic CI/CD for production releases | platform | medium |
| 20 | LLM-generated semantic cluster names | roadmap | low |
| 21 | AST decorator and annotation detection | roadmap | low |
| 22 | Git-diff impact analysis and refactor hints | roadmap | low |
| 23 | Fine-grained authorization | roadmap | low |

**`GITNEXUS-1` gates everything.** GitNexus is PolyForm Noncommercial 1.0.0 and a commercial
"Enterprise" tier exists. If internal hosting is not permitted, most of this backlog is void —
and `GITNEXUS-17` in particular reimplements what looks like a paid feature.

## Known-suspect items

Flagged while writing the backlog; confirm or refute each early, because they change sequencing.

- **#11 Incremental indexing** — embeddings are already incremental via SHA1 content hashing.
  This may be a much smaller job than it reads, or already done for the part that matters.
- **#5 Configurable graph backend** — LadybugDB may be shallowly or deeply coupled; nobody has
  counted the call sites. **Keeping LadybugDB is a legitimate outcome.**
- **#2 Multi-tenant baseline** — the whole shape depends on what `lbug.lock` actually
  serializes. If readers are not blocked by a writer, most of #5–#8 may be unnecessary.
- **#13 Microservice split** and **#18 Operator** — both are large, permanent maintenance
  surfaces written as "potentially". Treat as evaluations with a real option to decline.

## Map

| Path | What it is | Exists |
|---|---|---|
| `PROMPT.md` | Trigger prompt — review, ground, refine the backlog. | ✅ |
| `FINDINGS.md` | Per-item verdicts and the risk register. | ⬜ |
| `RESEARCH.md` | Codebase archaeology backing those verdicts. | ⬜ |
| `adrs/` | Open / plan-scoped ADRs. | ⬜ |
| `PLAN.md` | The sequenced, grounded backlog. | ⬜ |
| `QUESTIONS.md` | Open questions for the requester. | ⬜ |

Directories are created on their first real document.

## Where work is tracked

| Register | Location |
|---|---|
| Plane (human-first) | workspace `forks`, project **GITNEXUS** (`2e835738-6bf8-4120-a0d5-a0178d1130ef`) |
| beads (agent-first) | database **`gitnxs`** on `bd.agents.woven:3306` — **does not exist yet**; this repo has no `.beads/` |

Creating the `gitnxs` database is plumbing work owned by
`infrastructure/infrastructure` → `.claude/plans/gitnexus-delivery/`, because it must also be
added to `local.backup_databases` in `shared/kube/beads/backup.tf` or it gets no backup at all.

## Related

- `ARCHITECTURE.md`, `RUNBOOK.md`, `MIGRATION.md` — the fork's own docs, and the fastest route
  into the pipeline, storage layout and MCP tool surface.
- `~/.claude/rules/plans-and-docs.md` — governs this layout.
