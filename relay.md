# RELAY — Spec-Driven Incremental Development Framework

**Version 1.0** · Spec-Driven · Incremental Process Model · Track A: Staged Delivery / Track B: Parallel Lanes

## Overview

RELAY builds software through the incremental process model, decomposed into micro-tasks that individual agents complete in a single focused session. Agents are brilliant inside a bounded problem and unreliable across a sprawling one, so the framework's whole job is to make every problem bounded: interview until the picture is sharp, design until the picture is written down, then slice the design into specs an agent can hold in its head.

**Every handoff carries only what the next agent needs.**

---

## First Principle: The Context Budget

Every failure mode of agentic development—drift, hallucinated APIs, half-finished refactors—is a context problem. RELAY treats agent context like RAM on an embedded system: a hard budget, allocated deliberately.

### What a Builder agent reads

A feature spec, dependency contracts, and the constitution fit into ~35% of the context window, leaving headroom for code, tests, and reasoning.

**The context manifest** names every file the agent may read. If it isn't in the manifest, it doesn't exist. All remaining headroom goes to the actual work.

### What it never reads

- The full design document
- Chat history
- The entire codebase

Whole-project context produces confident agents with vague understanding. RELAY never asks an agent to "understand the system"—only to satisfy one spec.

### The Sizing Rule

**If a task's spec + manifest can't be read, implemented, and verified in one agent session, the spec is too big. Split the spec—never stretch the agent.**

---

## The Pipeline: Eight Phases, One Relay

Phases 0–4 are shared by both tracks and are dominated by **interviewing**—the user is the primary source of truth until the Design Document Spec is approved. Phase 5 onward is where the tracks diverge.

### Phase 0: Interview

**Agent:** Interviewer  
**Exit Gate:** User confirms playback

**Activities:**
- Multi-round structured interview: product vision & target users; functional requirements; non-functional requirements (scale, security, compliance, performance); constraints (tech preferences, hosting, budget, deadlines); explicit out-of-scope
- Interviewer distills, numbers every requirement (FR-001…, NFR-001…), and plays the full picture back to the user for line-by-line confirmation
- Unresolved ambiguity blocks the phase—guessing is forbidden

**Deliverable:** `docs/REQUIREMENTS.md`
- Problem statement
- Personas
- Numbered FRs & NFRs
- Constraints
- Out-of-scope
- Resolved open questions

### Phase 1: Architecture Gate

**Agent:** Architect + User  
**Exit Gate:** Track chosen & recorded

**Activities:**
- The Architect presents the fork—Monolith (staged delivery) vs Microservices (parallel lanes)—with a recommendation grounded in the interview: team shape, deployment target, scaling profile, operational appetite
- The user decides
- The decision, its rationale, and the rejected alternative are recorded ADR-style so no future agent relitigates it

**Deliverable:** `docs/ARCHITECTURE-DECISION.md`
- Context
- Decision
- Consequences
- Revisit criteria

### Phase 2: High-Level Design

**Agent:** Architect  
**Exit Gate:** User approves HLD in review interview

**Activities:**
- System architecture (components and their boundaries)
- Tech stack with rationale per choice
- Database design (entities, relationships, schema strategy, migration approach)
- Primary module/component map with one-line responsibilities
- Cross-cutting concerns designed once: auth, configuration, error handling, observability
- Design-review interview—the Architect walks the user through the HLD and revises until approved

**Deliverable:** `docs/design/HLD.md`
- Architecture diagram
- Stack table
- ERD
- Module map
- Cross-cutting decisions

### Phase 3: Low-Level Design

**Agent:** Detail Designer  
**Exit Gate:** Design Document Spec complete · (Track B) contracts frozen

**Activities:**
- One module at a time, sequentially even on the microservices track, so interfaces stay mutually coherent
- Internal logic
- Public API signatures
- Typed data structures
- Workflows/sequences
- Error taxonomy
- Communication standardization: in-process interfaces for the monolith; for microservices, one shared message envelope (`{id, ts, source, type, version, payload}`) plus a per-module contract

**Deliverable:** `docs/design/lld/<module>.md` per module  
`contracts/envelope.md` + `<module>.contract.md` (Track B)  
`docs/DESIGN.md` (index)

### Phase 4: Development Plan

**Agent:** Planner  
**Exit Gate:** User approves increment order

**Activities:**
- Builds the module dependency graph
- Slices it into increments sized against the context budget
- **Track A:** Ordered stages, each a vertical slice ending deployable—Stage 1 is always a walking skeleton
- **Track B:** Dispatch waves—Wave 0 shared scaffolding & contract stubs, Wave 1 leaf modules in parallel, Wave 2 dependents, and so on
- Every increment gets entry criteria, exit criteria (Definition of Done), and a demo script
- The `PROGRESS.md` ledger is initialized with one row per planned spec

**Deliverable:** `docs/PLAN.md`
- Dependency graph
- Increments table
- Demo scripts

`PROGRESS.md` skeleton

### Phase 5: Feature Specs

**Agent:** Spec Writer  
**Exit Gate:** Specs pass a buildability check—zero open questions

**Activities:**
- The next increment only—not the whole project—is decomposed into feature specs
- Each spec is completable in a single agent session
- A logging module might yield ten: scaffold, config, data-input, data-recording, rotation, query-api, retention, health, metrics, admin-ui
- Each spec carries §1 Purpose, §2 Logic Map, §3 Structure, §4 Interface, §5 Unit Tests, §6 Integration Map, §7 Definition of Done
- Plus a context manifest naming every file its Builder may read
- Just-in-time writing means facts from `FIXES.md` and prior §8 notes shape every new spec

**Deliverable:** `specs/<module>/<nn>-<feature>.spec.md`
- One baton per micro-task

### Phase 6: Build Loop

**Agent:** Builder → Verifier → Integrator  
**Exit Gate:** Stage gate (A) / module + wave gates (B)

**Activities:**
- **Track A—sequential relay:** Orchestrator dispatches one Builder per spec, in order. Builder works red→green→refactor inside its manifest, appends §8 notes + proof-of-execution. Verifier independently re-runs everything; FAIL sends spec + failure report to a fresh Builder. When a stage's specs are all verified, the Integrator runs the stage demo script against the live app—only a green gate opens Stage N+1
- **Track B—parallel lanes:** One lane per module, all at once; inside a lane the same sequential relay applies. Module gate: contract tests + the runs-alone soak (boots solo, listens, `/health` healthy for full uptime, idles gracefully). Wave gate: Integrator composes finished modules with consumer-driven contract tests

**Deliverable:**
- Verified code per spec
- Tagged runnable release + `docs/stages/stage-N-report.md` (A)
- Wave integration report (B)
- `PROGRESS.md` rows flipped to done

### Phase 7: Increment Review

**Agent:** Interviewer + Orchestrator  
**Exit Gate:** User accepts increment; plan updated

**Activities:**
- The working increment is demoed to the user
- The Interviewer returns for a feedback interview: what matched intent, what didn't, what the working software revealed about the next increment
- Feedback becomes Change Requests appended to `DESIGN.md` / `PLAN.md` changelogs—the incremental model's loop, with a paper trail
- Then the relay returns to Phase 5 for the next increment

**Deliverable:**
- Increment acceptance note
- Change Requests
- Updated `PLAN.md` for the next lap

---

## Two Tracks, One Goal

At the Architecture Gate the user chooses a track. Both produce the same document stack and the same micro-task discipline—they differ only in **how the build loop dispatches agents** and **what "done" means for an increment**.

### Track A: Monolith — Staged Delivery

**The Staged Delivery Rail**

One codebase, built as an ordered series of stages. Development is **strictly sequential—one agent at a time, never in parallel**. Each stage ends in a functional, deployable, demonstrable state before the next may begin.

**Example Stage Progression:**

1. **Walking skeleton** — app boots, config, routing, persistence wired, one end-to-end path works → ✓ SHIPPABLE
2. **Core domain** — the feature the app exists for, usable end to end → ✓ SHIPPABLE
3. **Second-priority features** — layered onto the working core → ✓ SHIPPABLE
4. **Polish & hardening** — every stage before this one already worked → ✓ SHIPPABLE

**Track A Rules:**

- **Sequential relay.** Exactly one Builder agent holds the baton at any moment. Its handoff note is the next agent's starting line
- **Stage gate.** A stage closes only when the Integrator runs the stage's demo script against the live app and captures proof-of-execution. No green gate, no Stage N+1
- **Stage sizing.** Stages are sized in feature specs, not weeks—small enough that the Spec Writer can draft the whole stage inside one context budget, keeping specs mutually consistent
- **Vertical slices.** Every stage cuts through UI → logic → storage so it's demonstrable, not a horizontal layer nobody can run
- **Visible progress.** Every spec and stage lives as one row in `PROGRESS.md`—the whole build is manageable at a glance, from kickoff to ship

### Track B: Microservices — Parallel Lanes on a Frozen Bus

**Parallel Lanes on a Frozen Bus**

Contracts are designed and **frozen before any code**. Then one agent lane is dispatched per module, all lanes in parallel. Lanes never talk to each other—**the contract is the only communication**.

**Example Parallel Module Lanes:**

```
◉ logging-service        GET /health → HEALTHY
  [01][02][03][04][05]

◉ auth-service           GET /health → HEALTHY
  [01][02][03][04][05][06]

◉ accounts-service       GET /health → HEALTHY
  [01][02][03][04]

⇅ MESSAGE BUS — ONE ENVELOPE SCHEMA · contracts/envelope.md · FROZEN BEFORE DISPATCH ⇅
```

**Track B Rules:**

- **Contract freeze gate.** The envelope schema and every module contract are user-approved and version-locked before the first Builder is dispatched. Changing a contract mid-wave is a formal Change Request, not an edit
- **Runs-alone requirement.** Every module must boot by itself, listen on its declared transport, and idle gracefully. A logging service with zero incoming messages keeps listening; its `/health` returns healthy for its entire uptime. No module may assume any peer exists
- **Small modules.** A module is one responsibility, one contract, and typically 4–12 feature specs. Bigger than that, the Planner splits it
- **Lane = mini-relay.** Inside each lane, specs are still built sequentially by single agents—parallelism is *between* lanes, never within one
- **Integration by waves.** Modules with no dependencies build first (Wave 1); the Integrator composes each wave with consumer-driven contract tests before the next wave dispatches

---

## The Document Stack

Documents are the framework's only memory. Agents come and go; the repo remembers. Each document is complete on its own—no artifact ever says "see previous conversation."

### Repository Layout

```
docs/
├─ REQUIREMENTS.md              ← Interview output, user-approved
├─ ARCHITECTURE-DECISION.md     ← Track decision, rationale, trade-offs
├─ DESIGN.md                    ← Index + changelog (master document)
├─ design/
│  ├─ HLD.md                    ← High-level design
│  └─ lld/
│     ├─ logging.md             ← Module detail: logic, API, structures, workflows
│     └─ <module>.md
├─ PLAN.md                      ← Dependency graph, increments table, demo scripts
├─ PROGRESS.md                  ← Ledger: spec-by-spec status (machine-readable)
├─ FIXES.md                     ← Discovered facts promoted from §8 notes
└─ CONSTITUTION.md              ← Shared coding rules, linting, conventions

contracts/                       ← Microservices only
├─ envelope.md                  ← The one message shape for the entire system
└─ <module>.contract.md         ← Per-module transport, messages, /health endpoint

specs/
└─ logging/
   ├─ 01-module-scaffold.spec.md
   ├─ 02-config-loading.spec.md
   ├─ 03-data-input.spec.md
   ├─ 04-data-recording.spec.md ← Feature spec (the atomic handoff unit)
   └─ <nn>-<feature>.spec.md
```

### Key Documents

**REQUIREMENTS.md** — Problem statement, personas, numbered FRs & NFRs, constraints, out-of-scope, resolved questions. Written by Interviewer, read by Architect and Planner.

**DESIGN.md (master index)** — Thin index: links, one-line module summaries, status, changelog of approved Change Requests. The weight lives in HLD.md + lld/*.md.

**HLD.md** — System architecture, tech stack + rationale, database design (ERD, schema & migration strategy), module map with responsibilities, cross-cutting concerns.

**lld/\<module>.md** — Per-module logic, API signatures, typed data structures, communication method, workflows, error taxonomy.

**PLAN.md** — Module dependency graph; increments table (Stage 1…N or Wave 0…N); per-increment entry/exit criteria and demo scripts; sizing notes; changelog for Change Requests from Phase 7 reviews.

**PROGRESS.md** — Machine-readable ledger: one row per feature spec (id, stage/wave, status, builder, verified_by, timestamp). Single-writer discipline (Orchestrator only) keeps it trustworthy. The entire project is auditable at a glance.

**CONSTITUTION.md** — Shared code rules, linting config, test patterns, deployment procedures, error handling conventions. Every Builder reads this.

**FIXES.md** — Discovered facts promoted from §8 Implementation Notes in finished specs. The Spec Writer reads this when drafting the next batch, so learnings flow forward.

---

## Anatomy of a Feature Spec

The feature spec is the baton itself—the smallest complete unit of work, and the only thing a Builder agent ever needs.

### Structure

Each feature spec lives at `specs/<module>/<nn>-<feature>.spec.md` and contains eight sections:

**Front Matter (YAML):**
```yaml
id: LOG-004
module: logging
title: data-recording
depends_on: [LOG-001 module-scaffold, LOG-003 data-input]
status: ready        # ready → in-progress → verify → done
est_sessions: 1
context_manifest:    # the ONLY files the Builder may read
  - specs/logging/04-data-recording.spec.md
  - contracts/logging.contract.md
  - src/logging/input.go
  - docs/CONSTITUTION.md
```

**§1 Purpose**

One paragraph: what this feature does and why it matters in the context of the module.

Example: "Persist validated log entries received from the data-input feature (LOG-003) to append-only storage, guaranteeing that an accepted entry is durably recorded exactly once, in arrival order, before acknowledgment is returned."

**§2 Logic Map**

Numbered behavior flow including every error path—the Builder implements this map, nothing more.

Example:
```
1. Receive Entry from input channel (LOG-003 hands off validated entries only)
2. Serialize Entry → envelope-conformant JSON line
   2a. Serialization fails → increment metric log_serialize_errors,
       emit ERR_SERIALIZE to dead-letter, DO NOT crash, continue loop
3. Append line to active segment file (fsync per batch of ≤64 or 50 ms)
   3a. Disk write fails → retry ×3 w/ backoff; on exhaust → ERR_STORAGE,
       set health = DEGRADED (not DOWN), buffer up to 10k entries in memory
4. Update write cursor; ack the entry to input channel
5. Segment ≥ 64 MB → signal rotation (owned by LOG-005; only signal here)
6. Loop. Empty channel is a normal state—block on receive, never busy-poll.
```

**§3 Structure**

Files to create, data structures (field-level, typed—no ambiguity left for the agent to invent).

**§4 Interface**

Every public function with full signature, parameters, returns, and error contract.

**§5 Unit Tests**

Enumerated test cases, written to fail first (red → green → refactor). The Verifier re-runs exactly this list.

**§6 Integration Map**

How the rest of the system uses this feature—the only section other agents ever read from this spec:

- **Consumes:** What channels or APIs this feature receives from
- **Exposes:** Public methods and contracts other modules call
- **Emits:** Signals, events, or messages this feature produces
- **Message examples:** Sample envelope-conformant payloads
- **Never assumes:** What this feature does NOT depend on

**§7 Definition of Done**

Checklist of completion criteria: all tests pass, no changes outside declared files, code is formatted/linted, CONSTITUTION rules honored, Implementation Notes written, PROGRESS.md row flipped to `verify`.

**§8 Implementation Notes**

Appended by the Builder at handoff—the baton's payload. Deviations from spec (with reasons), facts discovered during implementation, and the captured proof-of-execution output live here. The Verifier reads it first; anything of system-wide relevance is promoted to `FIXES.md` so future specs are written against reality, not hope.

---

## The Agent Roster

Nine roles. Each has a fixed diet (what it reads) and a fixed deliverable (what it hands off). No agent ever does another agent's job.

### Orchestrator

**Reads:** `PLAN.md` · `PROGRESS.md`  
**Delivers:** Task dispatches with context manifests; the only writer of `PROGRESS.md`  
**Rule:** Routes work; never produces design or code itself

### Interviewer

**Reads:** The user, exhaustively—vision, users, features, constraints, scale, compliance  
**Delivers:** `REQUIREMENTS.md` after a confirmed "here's what I heard" playback  
**Rule:** Asks and distills; never proposes solutions

### Architect

**Reads:** `REQUIREMENTS.md`  
**Delivers:** `ARCHITECTURE-DECISION.md` (with track recommendation) · `HLD.md`, revised through a design-review interview  
**Rule:** Presents the track fork; the user decides

### Detail Designer

**Reads:** `HLD.md` + one module's scope at a time  
**Delivers:** `lld/<module>.md` · module contract + shared envelope (Track B)  
**Rule:** Designs modules sequentially so contracts stay mutually coherent

### Planner

**Reads:** `DESIGN.md` index (HLD + LLD summaries)  
**Delivers:** `PLAN.md`—dependency graph, stages or waves, entry/exit criteria—and the `PROGRESS.md` skeleton  
**Rule:** Sizes every increment against the context budget

### Spec Writer

**Reads:** `PLAN.md` + one module's LLD  
**Delivers:** Feature specs (§1–§7) for the *next* stage or lane only—written just-in-time so `FIXES.md` learnings flow in  
**Rule:** Every spec must be buildable with zero questions

### Builder

**Reads:** Exactly one spec's context manifest  
**Delivers:** Code + tests via red→green→refactor, plus §8 Implementation Notes and proof-of-execution  
**Rule:** Fresh session per spec; never claims done without captured output

### Verifier

**Reads:** The spec + the Builder's diff  
**Delivers:** Independent PASS/FAIL with its own test run. FAIL → failure report dispatched to a *fresh* Builder, never a stale session  
**Rule:** Trusts nothing it didn't execute itself

### Integrator

**Reads:** Integration maps (§6) of the specs in a stage or wave—never their internals  
**Delivers:** **Track A:** The stage gate—demo script run against the live app, tagged release, stage report. **Track B:** Wave composition + consumer-driven contract tests + standalone soak checks  
**Rule:** An increment isn't done until it runs

---

## The Constitution: Eight Handoff Rules

These are invariant across both tracks. They exist because every one of them, when broken, is a documented way agent projects die.

### 1. The manifest is the world

An agent may read only the files its context manifest names. Curiosity beyond it is a defect, not diligence.

### 2. Specs are the only channel

Agents communicate through documents, never through conversation. If it matters, it's written down; if it isn't written down, it didn't happen.

### 3. Every artifact stands alone

Any document can be handed to a brand-new agent with zero history and be fully actionable.

### 4. Done requires proof

Every completion claim ships with captured command output—the actual test run, the actual boot log, the actual health check.

### 5. Fresh agent on failure

A failed verification dispatches a new Builder with the spec plus the failure report. Stale sessions carry stale assumptions.

### 6. Discovered facts get promoted

Anything learned mid-build lands in §8, and system-wide facts move to `FIXES.md` so the Spec Writer drafts the next increment against reality.

### 7. One writer for progress

`PROGRESS.md` is a ledger with a single writer (the Orchestrator). Every spec is one row: id, stage/wave, status, verified_by, timestamp. The whole project is auditable at a glance.

### 8. Change is formal

Post-approval edits to design docs or contracts happen through a Change Request appended to the document's changelog—the incremental model's feedback loop, with a paper trail.

---

## Accessibility & Safety

### Contrast & Legibility

- Body text: only ink-on-base, ink-on-paper, or base-on-ink. Accents are for surfaces, large/bold type, and UI chrome
- Never convey state by text color alone: pair with text, icon, or weight

### State Indication

- Validation states (error, success) require text + icon + color, not color alone
- Focus states have their own explicit treatment, never just a shadow
- Hit targets are sized by padding, not border weight

### Documentation

Every spec's §6 Integration Map must be clear enough that a peer agent can use the feature without reading §2–§5. If it can't be explained in §6, the interface isn't clear enough.

---

## Quick Reference: How to Run RELAY

### For Monolith (Track A)

1. **Phases 0–4:** Interview, architecture decision, HLD, LLD, plan (shared with Track B)
2. **Phase 5:** Spec Writer drafts Stage 1 feature specs
3. **Phase 6:** Builder → Verifier relay on each spec; Integrator gates the stage when all specs pass
4. **Phase 7:** Demo Stage 1, gather feedback, update plan
5. **Loop:** Spec Writer drafts Stage 2; relay continues sequentially

### For Microservices (Track B)

1. **Phases 0–4:** Same as Track A, but Detail Designer designs all module LLDs + contracts upfront
2. **Phase 5:** Spec Writer drafts Wave 0 (scaffolding) + Wave 1 (leaf modules with no dependencies)
3. **Phase 6:** All Wave 1 modules dispatch in parallel, each with its own Builder → Verifier relay. Integrator runs contract tests + soak checks before unlocking Wave 2
4. **Phase 7:** Demo Wave 1, gather feedback, update plan
5. **Loop:** Spec Writer drafts Wave 2 (modules that depend on Wave 1); relay continues with parallel dispatch

---

## Why RELAY Works

**Context problems are solved by making every problem bounded.** Feature specs are bounded by their context manifest. Stages are bounded by their entry/exit gates. Lanes are bounded by frozen contracts. No agent is ever asked to "understand the system"—only to satisfy one spec, with proof.

**Progress is visible and manageable end-to-end.** PROGRESS.md is a single ledger row per spec. The entire build is auditable at a glance. When someone asks "how's it going?" the answer is one grep.

**Learning flows forward.** §8 Implementation Notes capture deviations and discovered facts. System-wide learnings move to FIXES.md. The Spec Writer drafts the next batch against reality, not hope. This closes the feedback loop without requiring anyone to remember.

**Handoffs are designed, not hoped for.** Every agent knows exactly what it reads, what it writes, and what "done" looks like. The Orchestrator's job is pure routing—no surprises, no scope creep, no ambiguous interfaces between agents.

---

## Glossary

**Baton:** The spec + Implementation Notes handed from one agent to the next. Contains only what the next agent needs.

**Context Budget:** The fixed amount of context window an agent gets for a task (spec + manifest ~35%, reasoning/work ~65%).

**Context Manifest:** The list of files a Builder may read for a given spec. Everything else doesn't exist for that agent.

**Definition of Done:** The §7 checklist proving a spec is complete, verified, and ready to promote.

**Increment:** A stage (monolith) or wave (microservices) — a set of feature specs that close together with a single gate.

**Integration Map:** The §6 section of a spec—how other agents use this feature without reading its internals.

**Lane:** A parallel module development track (microservices only). Sequential relay happens inside each lane; parallelism is between lanes.

**Proof-of-Execution:** Captured command output (test run, boot log, health check) proving a claim of completion.

**Stage Gate:** The moment the Integrator runs the demo script against a live app and captures proof the stage is shippable.

**Wave:** A set of modules deployed together (microservices only). Wave 0 is scaffolding, Wave 1 is leaf modules, Wave 2 depends on Wave 1, etc.

---

*RELAY framework v1.0 · Spec-driven · Incremental process model · Small specs, clean handoffs, every agent gets exactly the context it needs and nothing it doesn't.*
