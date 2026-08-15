# agentic-runs

Immutable Git-anchored history of every agentic run the P-43
N-agent pipeline substrate executes.

This repo is the **Phase 5 Git anchor** for P-43 (`Agentic AI
Team workflow`). Per D-417, the anchor lives in a dedicated repo
(not in `notes-db` and not in any SSoT team timeline) because
run history is structurally distinct from substrate code and from
human team state — mixing them creates coupling that compounds
over time.

## What this repo is

For every run, the substrate commits a self-contained artifact
tree to a dedicated commit. The commit SHA is the run's
**identity**; the contents are the run's **proof**. Anyone
who can clone this repo can answer:

- *What did the agents do?* — the `envelopes.jsonl` is the
  handoff chain.
- *What did the substrate observe?* — the `audit_events.sql`
  dump is the SQL slice of the audit_event table for the run.
- *What was the run state at every checkpoint?* — the
  `state.json` is the state-machine snapshot.
- *What did the run produce?* — the `output/` directory holds
  the deliverable files (the report HTML, the index-card
  excerpt, the structured brief echo).

The repo is pipeline-agnostic — it accepts runs from any N-agent
pipeline registered in `~/agentic/pipelines/<slug>.yaml`. The
first demonstrated pipeline was the 3-agent AI-hype demo
(researcher-firecrawl → editor-magazine → coder-html); the
second is the 4-agent code-review pipeline
(coder-minimax → preflight-runner → reviewer-grok →
coder-minimax/commit). The repo shape is identical for both.

## What this repo is NOT

- It is **not** the agent code. That lives in `~/agentic/`
  (and lands in `notes-db` as a substrate module per the build
  plan; Day-1 does not place it).
- It is **not** publishable output. The publishable output of
  a run is `~/research/reports/<slug>/report.html` + the
  `~/research/index.html` card — that path is the human-facing
  surface. The copies in `output/` here are the **provenance
  copy** for the audit trail, not the canonical delivery.
- It is **not** a human-team artifact. SSoT (the team's sprint
  ledger) lives in its own repo. An agentic run is a machine
  event, not a human event, and does not belong in the team
  timeline.

## Repository layout

```
~/agentic-runs/
├── README.md                       ← this file
├── .gitignore                      ← top-level ignore
└── runs/
    ├── .gitignore                  ← per-run generated artifacts
    ├── .locks/                     ← per-run lockfile directory (committed-on-cleanup convention)
    │   └── <run_id>.lock           ← created at run start, removed at run end (or kept for forensics)
    └── <run_id>/
        ├── run.json                ← run metadata (run_id, topic brief, started_at, status)
        ├── state.json              ← state-machine snapshot (PENDING/EXPLAINED/FAILED + last transition)
        ├── envelopes.jsonl         ← append-only handoff envelope chain (one JSON object per line)
        ├── audit_events.sql        ← SQL dump of audit_event rows for this run (sqlite3 .dump fragment)
        ├── commit.sha              ← this directory's commit SHA (written after each commit)
        ├── commit.msg              ← the commit message used for the immutable anchor commit
        └── output/
            ├── report.html         ← provenance copy of the deliverable report
            ├── index_card.html     ← provenance copy of the 200-word index-card excerpt
            ├── topic_brief.json    ← the structured brief that drove the run
            └── env_snapshot.json   ← the env-parameter snapshot Agent A saw (per OQ#4)
```

Each run lives in its own directory; the directory is the unit
of commit. One run == one commit (with the
`commit.sha` file written after the commit so the agent
transport can later cite the identity without re-deriving).

## Commit discipline

- **One commit per run.** The commit message references the
  `run_id` and the brief. Format:
  ```
  P-43 run <run_id> [<status>] <topic-slug>

  <one-line summary of what the run did>

  <audit_event_ids if run succeeded; reason if run failed>
  ```
- **FAILED runs are committed.** Per the P-42 decline-and-log
  philosophy, the trail of restraint is the value. A run that
  hit the 3-attempt cap is committed with `status=FAILED` and
  the reason recorded in the commit body.
- **The commit is the identity.** Once committed, the run
  directory is read-only on the working tree. Subsequent edits
  to a run are made by committing a new file (e.g.,
  `corrections/<run_id>.md`), never by amending the run's
  commit.

## Cross-references

- **D-417** — the decision that this repo exists.
- **P-43 parking-lot entry** — `~/PARKING_LOT.md`, the entry
  appended 2026-08-13 (the five-phase pipeline design this
  repo anchors).
- **P-42 parking-lot entry** — `~/PARKING_LOT.md` line 4747,
  the substrate this repo writes to.
- **P-43 Day-1 plan** — `~/notes/P43-DAY-1-PLAN-2026-08-13.md`,
  the spec the build phase will execute.
- **Substrate write target** — `~/notes_data/working.db` (the
  `audit_event` table). The `audit_events.sql` files in
  `runs/<run_id>/` are SQL dumps of the substrate rows for
  this run, not the canonical storage.

## Status

D-417 locked 2026-08-13. Repo bootstrapped with 2 commits
(the structure and the README split across two commits so they
read as distinct history steps) and pushed to GitHub at
<https://github.com/falken76/agentic-runs> (public). The exact
SHAs are visible via `git log --oneline` or
<https://github.com/falken76/agentic-runs/commits/main>.

Zero runs committed. Build phase is Day-2+ (D-423+).
