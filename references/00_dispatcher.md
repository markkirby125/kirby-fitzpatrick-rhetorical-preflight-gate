# Rhetorical Preflight Gate — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [A.I. Mistakes Writers Must Stop Making](https://www.youtube.com/watch?v=3kf9rRztJgA)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Situational Network

Most AI failures are not language failures. The prose is grammatical, the code compiles, the identifiers follow convention, the diff is clean. The artifact is simply an answer to a question nobody asked, delivered to a reader who was never in the room, executed at a scope that was never authorized. Fitzpatrick's lecture names the root cause precisely: the generator — human or language model — treats writing as **token production**, a surface operation performed *on text*, instead of treating it as a **response to a situation**, a deliberate intervention into a network of people, expectations, and consequences.

Surface fluency is the failure mode that hides the failure mode. A model that can produce plausible code on demand will also produce plausible *unrequested* code on demand, and both arrive in the same confident register. The lecture's diagnostic sequence — substance before surface, model the system before emitting tokens, navigate the codebase before generating — collapses into one operational rule: **no generation before situation mapping.**

**The Rhetorical Preflight Gate** installs that rule as a hard gate placed *before* the first token. Flight crews do not negotiate with aerodynamics after rotation; they run a checklist while the aircraft is still at the gate, when a discovered fault costs a taxi back instead of an emergency descent. Software changes share that property exactly: the cost of a boundary error discovered at minute zero is one clarifying question; at minute ninety it is a 400-line diff; after merge it is a revert, a changelog entry, and a reviewer who trusts the next PR less.

Seven load-bearing definitions:

1. **Situational Network (SN)** — the four-node map that every artifact is a response to: **{Trigger, Envelope, Blast Radius, Reviewer}**. Code is not a text to be improved; it is a move inside a network. A change that is correct in isolation and wrong in the network fails in review, and network-blindness is the defect this gate exists to catch.
2. **Trigger (T)** — the concrete occasion that made the artifact necessary: the incident ID, the support ticket, the metric crossing a threshold, the decision deadline. A trigger is an *event*, never a preference (*"the code looked dated"* has no trigger and therefore no artifact).
3. **Task Envelope (E)** — the authorized boundary, written as **verb + object + acceptance criterion + exclusion list**. The exclusion list is not decoration; it is the only part of the envelope that constrains a generator's improvisation. An envelope with an empty exclusion list is an unbounded envelope wearing a task's clothes.
4. **Blast Radius (B)** — everything whose behavior or meaning changes when the artifact lands: call sites, exported symbols, contracts, schemas, config keys, deploy units, docs, on-call humans. Blast radius is *measured before editing*, because after editing it is discovered by the build.
5. **Reviewer Expectation Horizon (R)** — the set of questions the actual named reviewer will ask and where they will look first. The reviewer is a specific person with a specific queue, not an abstract "best practices" oracle. A diff that forces them to hold two or more independent decisions at once is rejected or rubber-stamped — both outcomes remove the evidence from the review.
6. **Unrequested Refactor (UR)** — any hunk whose rationale lives in the author's taste rather than in the request: renames, reformatting, docstring sweeps, dependency swaps, "modernization", opportunistic extractions. UR is not a style crime; it is a **review tax** — it converts a two-decision review into a five-decision review and hides the actual fix inside the noise. It also makes rollback all-or-nothing.
7. **Preflight Card** — the four-line artifact emitted *before* the diff (Trigger / Envelope / Blast Radius / Reviewer + acceptance). The card is the protocol's deliverable; the code is its consequence. If the card cannot be written, the task is not yet understood.

Two derived measures make the gate auditable rather than aspirational:

- **Scope Ratio (SR)** = authorized lines ÷ total lines in the diff. A bug fix should hold **SR ≥ 0.9**. An SR below 0.5 means the artifact is a rewrite wearing a bug-fix label, and the reviewer is auditing the wrong object.
- **Decision Count (DC)** = independent decisions a reviewer must accept in one unit. Target **DC = 1**. Every additional decision multiplies the probability that the unit is approved for social reasons.

```text
[ANTI-PATTERN: Generate First, Situate Never]   ->  the coat-of-paint diff
  request: "paging returns one row too many"          (Trigger read: never)
       |
       v
  +--- patch.diff  (413 lines) ----------------------------------------------+
  | fix off-by-one .................  2 lines   <- the only authorized change |
  | rename Page -> CursorPage ..... 122 lines   <- unrequested                |
  | reformat indentation .......... 226 lines   <- unrequested                |
  | add docstrings ................. 58 lines   <- unrequested                |
  | swap JSON library ............... 5 lines   <- unrequested, unmeasured    |
  +---------------------------------------------------------------------------+
  Scope Ratio   0.005          Decision Count 5 (bundled into one review unit)
  Blast radius  unknown - 5 files touched, 2 exported symbols renamed
  Reviewer      holds 5 decisions at once -> stall for two days, then approve
  Rollback      all-or-nothing; the fix cannot be reverted without the rename
  Result        the fix was never reviewed in isolation; no one can prove it works

[RHETORICAL PREFLIGHT GATE]   ->  same request, five lines of preflight first
  TRIGGER       INC-4477: GET /v2/orders?limit=N returns N+1 rows (ticket 8812)
  ENVELOPE      verb:    fix
                object:  paging boundary at handler.go:88
                accept:  limit=N returns exactly N rows
                         + regression test that fails before, passes after
                exclude: renames, formatting, docstrings, dependency changes
  BLAST RADIUS  3 call sites (rg "listOrders" -> 3 hits), 1 exported symbol,
                no schema or config surface, no deploy-order coupling
  REVIEWER      @dana (service owner) - checks: wire contract preserved?
                does the test prove the boundary? is revert one commit?
  ---------------------------------------------------------------------------
  +--- patch.diff  (11 lines) -----------------------------------------------+
  | fix off-by-one .................  2 lines   (authorized by ENVELOPE)      |
  | regression test ................  9 lines   (authorized by accept)        |
  +---------------------------------------------------------------------------+
  Scope Ratio 1.00             Decision Count 1
  Deferred, tracked, not shipped: rename (#451), formatter pass (#452),
  docstring sweep (#453) - each proposed as its own review unit.
  Result: approval is evidential; revert is one commit; the fix is legible.
```

Why this binds harder in software than in prose: an artifact's readers arrive **later, colder, and under time pressure** — the on-call engineer at 03:00 with three candidate commits, the new hire in week one, the auditor, the author's own self in six months. Prose that overreaches wastes a reader's minutes. Code that overreaches rewrites a blast radius nobody measured, converts a review into a negotiation, and destroys the reviewer's ability to attribute a regression to a change. The preflight costs five lines. Its absence costs a revert plus the diff that was never audited.

**The gate is not a stylistic preference and not a ban on refactoring.** It is a sequencing constraint: *situate, then bound, then generate, then re-audit the diff against the bound.* Refactoring remains fully available — as its own artifact, with its own trigger, its own envelope, and its own reviewer. What the gate forbids is smuggling it past review inside an unrelated fix.

---

## 2. Core Transformation Protocols

1. **Answer the four preflight axes before emitting a single token of code or documentation.** Trigger, Envelope, Blast Radius, Reviewer. Any axis that cannot be answered becomes an explicit `UNKNOWN` with an owner and a question — never a silent assumption. One clarifying round-trip is cheaper than one unrequested refactor.

2. **Write the envelope with a mandatory exclusion list.** Verb + object + acceptance criterion + **what is explicitly not in scope**. The exclusion list is where the boundary becomes real, because it is the only clause that speaks to a generator's improvisation. No exclusion list, no preflight.

3. **Apply the Inverse Diff Test to every hunk.** For each hunk, name the envelope clause that authorizes it. An unauthorized hunk is not reviewed, not softened, and not kept "since it's already written" — it is moved to the deferred list and proposed as its own review unit.

4. **Measure blast radius before editing, with a command.** `rg` the symbol, list the call sites, count the exported surfaces, name the contract files. State the number in the card. "Small change" is a feeling; "3 call sites, 1 exported symbol, 0 schema changes" is a measurement.

5. **Separate the three change layers and never ship them in the same diff.** *(a)* the requested change, *(b)* mechanical fallout required to keep the build green, *(c)* opportunistic improvements. Layer (c) never rides inside layer (a). Layer (b) is allowed only when the build or test suite fails without it — verify that claim, do not assume it.

6. **Optimize for the reviewer's queue, not for the author's taste.** Minimize Decision Count. One review unit carries one decision; a second decision is a second PR. A reviewer who must hold five decisions at once is not reviewing, they are adjudicating.

7. **Gate cosmetic work behind semantic gates.** Formatting, docstrings, comment sweeps, and renames are permitted only *after* correctness is proven and the test gate is green — never as the opening move on unverified code. "Coat of paint" applied to broken code makes the breakage harder to see, not less broken.

8. **Ground before generating.** Read the actual callers, types, tests, and config rather than pattern-completing from the request text. Plausibility is not grounding; a signature that *looks* right and is not present in the repository is the single most expensive AI-generated defect, because it reads as idiomatic.

9. **Never silently expand scope.** When the boundary audit surfaces an adjacent problem, surface it as a named proposal with a cost estimate and let the requester own the decision. Reported findings are contributions; unilaterally fixed findings are unauthorized changes with a good story attached.

10. **Declare what you did not do.** Every PR body carries an explicit "Not in this change" list. It must be *true* — verified by re-reading the final diff against the exclusions — because a false exclusion list is worse than none: it teaches reviewers to trust a claim they never checked.

11. **Re-audit after generation, not only before.** The gate is run twice: once to bound the work, once against the finished diff. Post-generation drift is normal — a "one-line" fix turns into a signature change during implementation — and the second pass is what catches it while it is still cheap.

12. **Escalate the boundary conflict instead of resolving it yourself.** When the request cannot be satisfied inside its envelope, that is a *finding about the request*, delivered as a preflight card with the conflict named plus two options. Never resolve an authority question by writing more code.

### 2.1 The preflight card: four axes, four required answer shapes

| Axis | Question the gate asks | Required answer shape | Fails preflight |
|---|---|---|---|
| **Trigger** | What event made this artifact necessary, and what changes in the world when it lands? | Event + reference: incident ID, ticket, failing test name, metric + window, deadline | *"the code looked dated"*; *"good hygiene"*; *"while I was in there"* |
| **Envelope** | What is the authorized verb, object, acceptance criterion — and what is explicitly excluded? | One verb + one object + a checkable acceptance criterion + an exclusion list | An acceptance criterion no test can express; an empty exclusion list |
| **Blast Radius** | What else changes or breaks, and how was that measured? | Named call sites, exported symbols, contract/schema/config surfaces, deploy coupling, plus the command that found them | *"shouldn't affect anything"*; *"it's internal"*; a blast radius argued from intuition |
| **Reviewer** | Who reads this, what will they check first, and what counts as approval? | A named person or role + their first three checks + the explicit accept/reject condition | *"the team"*; *"whoever's on rotation"*; no stated accept condition |

**Field rule.** An axis answered with a feeling is an `UNKNOWN`. An `UNKNOWN` is not a license to proceed; it is a question with a name attached to it, asked before generation rather than after review.

### 2.2 Transformation table: anti-patterns and clean replacements

| Anti-Pattern (as emitted) | Rhetorical diagnosis | Clean Replacement |
|---|---|---|
| *"While I was in there, I also renamed `Page` to `CursorPage` across the repo."* | Scope leak past the envelope; the real fix becomes unauditable inside the rename | *"Fix only: `handler.go:88`. Rename deferred and tracked as #451 — 122 lines, 2 exported symbols, a separate review unit."* |
| *"Cleaned up formatting and added docstrings while fixing the bug."* | Cosmetic layer applied before the semantic gate; hides the breakage | *"Semantic fix + regression test only. Formatter and docstring pass deferred until the fix is green and merged (#452, #453)."* |
| *"I assumed the caller expects `null`."* | Ungrounded assumption substituted for a boundary fact | *"Preflight UNKNOWN resolved: `parseOrder` returns `null` on empty body (`handler.ts:88`). Assumption removed; behavior is read, not guessed."* |
| *"Refactored the module for clarity."* | Rationale lives in author taste; the trigger does not authorize it | *"Envelope is `fix off-by-one in paging`. No refactor. Clarity change proposed with cost and rationale, owned by the requester."* |
| *"Should work for other callers too."* | Blast radius asserted, never measured | *"Blast radius: `rg listOrders` → 3 call sites, all updated; 1 exported symbol signature unchanged; 0 schema/config changes."* |
| *"Modernized to the new internal pattern."* | No trigger exists for the change; the occasion is invented | *"Trigger is INC-4477 (timeouts). No modernization in scope; modernization logged as follow-up #454 with a size estimate."* |
| *"Fixed the bug and added a feature flag for future work."* | Two decision layers in one unit; DC = 2 | *"Split: PR A = bug fix (DC 1). PR B = flag, deferred with a trigger of its own; a flag whose roll-out is untriggered is unreviewable."* |
| *"Small change, mostly cleanup."* | Unquantified claim; the reader cannot size the risk | *"11 lines, SR 1.00, DC 1: 2-line fix + 9-line regression test. No cleanup in this change."* |
| *"I also updated the docs to match."* | Layer (b) claimed without a build failure proving necessity | *"Docs updated because `docs/api.md:40` documents the old boundary and now contradicts the code — required by the acceptance criterion."* |
| *"Touched 5 files, should be safe."* | Blast radius used as a count instead of a classification | *"5 files: 1 source fix, 3 call-site updates required by the type change, 1 test. Exported surface unchanged."* |

### 2.3 Scope-leak taxonomy

| Leak class | Typical shape | Why it must be its own artifact |
|---|---|---|
| **Cosmetic** | formatter run, indentation, comment/docstring sweep, import sort | Inflates the diff past the reviewer's patience for the real change |
| **Naming** | `Page` → `CursorPage`, `Manager` → `Service`, "consistent naming" | Touches exported surfaces; renames outlive the fix and complicate blame |
| **Structural** | extraction, file split, layer move, "cleaner" module boundaries | Moves code the reviewer just finished understanding; hides behavior change |
| **Dependency** | swapping a library, adding a helper, upgrading a version | Introduces an unreviewed failure mode under a fix's justification |
| **Feature** | flags, config knobs, "future-proofing", unused parameters | Adds surface the trigger never requested and no test covers |
| **Documentation** | README, ADR, docstring edits unrelated to the acceptance criterion | Dilutes the evidence trail; the review comment lands on the wrong file |

### 2.4 Failure diagnostics

| Symptom in review | Preflight diagnosis | Fix |
|---|---|---|
| *"What does this PR actually fix?"* | Envelope object absent; the diff is a file list | Write the envelope first: verb + object + acceptance criterion |
| *"Can we split this?"* | DC > 1; multiple layers in one unit | Extract layer (c) into its own PR; the requester keeps the decision |
| *"Why is this file touched?"* | A hunk with no authorizing clause | Run the Inverse Diff Test; delete or defer the hunk |
| *"Is this a rename PR or a fix PR?"* | Naming leak in a fix's clothing | Revert the rename out of this unit; re-land it under its own trigger |
| Reviewer approves without reading the fix | The fix is buried inside 400 unrelated lines | Restore SR ≥ 0.9; the noise is the reason the review failed |
| *"Did you check the other callers?"* | Blast radius unmeasured | Report the `rg` count and the files, not a reassurance |
| Review stalls on a design argument | A premise entered the diff without a trigger to authorize it | Move the premise into an RFC; keep the diff inside the envelope |
| Post-merge regression traced to an "innocent" extra hunk | Scope leak shipped as mechanical fallout | Layer (b) requires a proving build failure; else it is layer (c) |
| Author answers scope questions verbally, edits nothing | Decisions not patched back into the artifact | The exclusion list and the envelope belong in the PR body |

**Related dispatchers.** Give the artifact a verdict-first shape once its boundary is fixed with the [Here's Why Inversion](../../kirby-fitzpatrick-heres-why-inversion/SKILL.md); ground every claim in a falsifiable baseline through the [3-Part Proposal Engine](../../kirby-fitzpatrick-3part-proposal-engine/SKILL.md); audit the emitted document from zero context with the [Cold Reader PR Auditor](../../kirby-fitzpatrick-cold-reader-pr-auditor/SKILL.md); strip zero-information scope words (*cleanup*, *modernize*, *basically*) with the [Lexical Anti-Bloat Filter](../../kirby-fitzpatrick-lexical-anti-bloat-filter/SKILL.md); adapt the same envelope for three reviewer archetypes via the [3-Persona Stakeholder Framer](../../kirby-fitzpatrick-3persona-stakeholder-framer/SKILL.md).

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews — The Boundary Audit

The reviewer is the last line of defense against a scope leak, and the reviewer's own comment is an artifact subject to the same gate. Two laws govern this scenario. First, **the envelope is reconstructed from the request, never inferred from the diff** — a diff is evidence of what was done, never evidence of what was authorized. Second, **the reviewer must not leak scope either**: *"while you're in there, also fix X"* is the same defect wearing reviewer authority, and it is how one clean PR becomes three entangled ones.

**Boundary audit procedure:**

1. Read the request, the ticket, or the incident record first. Write the envelope: verb, object, acceptance criterion, whatever exclusions the requester implied.
2. Open the diff and classify each hunk: **IN-ENVELOPE** (authorized directly), **MECHANICAL** (required for the build/tests to pass), **OUT-OF-SCOPE** (rationale is author taste).
3. Verify the mechanical class rather than trusting the label: does the build actually fail without it? A hunk that cannot prove necessity is OUT-OF-SCOPE with a good story.
4. Never review OUT-OF-SCOPE hunks on style. Reviewing them legitimizes them. Return them as *one* request: extract, defer, or track — do not itemize twenty style notes on code that should not be in the diff.
5. State the acceptance condition once, in the reviewer's own words, so the author knows what closes the review.
6. If a genuine adjacent defect exists, file it as its own proposal with a trigger and a cost. Boundary hygiene is not the same as ignoring problems; it is refusing to launder them into an unrelated review.

**Before — the reviewer launders the leak:**

> A few things here. We usually don't camelCase in this package, and I'm not sure about the JSON library swap — can you use the house wrapper? Also this whole file could be split up. And I think the off-by-one is fixed? Hard to tell with all the reformatting honestly.

Diagnostics: the scope leak is *accepted* as a fait accompli; the reviewer now spends authority on style instead of the one item that matters; the acceptance condition for the fix is never stated; and the request ("use the house wrapper") expands the envelope again.

**After — the boundary audit, stated as one decision:**

```markdown
**Envelope check (from INC-4477):** fix the paging boundary at `handler.go:88` so
`limit=N` returns exactly N rows, with a regression test. Nothing else.

**Hunk classification**
| Hunk | Class | Authorizing clause / necessity proof |
|---|---|---|
| `handler.go:88` off-by-one | IN-ENVELOPE | verb: fix, object: paging boundary |
| `handler_test.go` new case | IN-ENVELOPE | acceptance: limit=N returns N |
| `Page` → `CursorPage` (122 ln) | OUT-OF-SCOPE | no clause; rationale is naming preference |
| reformat (226 ln) | OUT-OF-SCOPE | no clause; formatter run, revertible |
| JSON library swap (5 ln) | OUT-OF-SCOPE | no clause; changes an unreviewed failure mode |

**Blocking:** split out the three OUT-OF-SCOPE hunks. With them removed, the
remaining 11 lines are reviewable in one pass and revert is one commit.
The rename is a real improvement — please re-land it as its own PR (122 lines,
2 exported symbols touched). I will review it on its own merits today.

**Non-blocking, filed not requested:** file split → #455 (owner: @dana).
**Accept condition:** `limit=N` returns N rows; the new test fails on `main`
and passes on this head; `rg "listOrders"` shows 3 call sites, all unchanged.
```

**Acceptance rule.** A review is complete when the envelope is stated, every hunk is classified, and the accept condition is explicit. A blocking comment names the hunk class and the authorizing clause it lacks. A reviewer's suggestion that adds scope is filed as a proposal, never issued as a condition of approval.

### 3.2 PR Descriptions — The Preflight Card as the PR Header

The PR body is the artifact's durability layer: the diff records mechanics, the body records *authorization and boundary*. Six months later the diff is still readable and the reason it was allowed to exist is gone. The Preflight Card belongs at the top of the body, before the evidence, so a reviewer can falsify the scope in fifteen seconds without reading a single hunk.

```markdown
## Preflight
| Axis | Answer |
|---|---|
| **Trigger** | INC-4477 — `GET /v2/orders?limit=N` returns N+1 rows; reported 4× since 2026-03-01 (ticket 8812) |
| **Envelope** | verb: **fix** · object: paging boundary `handler.go:88` · accept: `limit=N` returns exactly N rows + a regression test that fails on `main` |
| **Exclusions** | no renames · no formatting · no docstring sweep · no dependency changes · no file restructuring |
| **Blast radius** | `rg -n "listOrders"` → 3 call sites, all unaffected (signature unchanged); 1 exported symbol, unchanged; 0 schema, 0 config, 0 deploy-order coupling |
| **Reviewer** | @dana (service owner) — checks: (1) wire contract preserved, (2) test proves the boundary, (3) revert is one commit |
| **Accept** | 11 lines, SR 1.00, DC 1; test red on `main @ 4f9c1ab`, green on head `3a77e02` |

## What changed
`slicePage` clamps the upper bound to `start+limit` instead of `start+limit+1`.

## Evidence
| Claim | Mechanism | Artifact |
|---|---|---|
| `limit=50` → 50 rows | inclusive end bound corrected (`handler.go:88`) | `TestListOrdersLimit` — red on `main`, green on head |
| contract unchanged | response envelope built by the same assembly path | `orders.v1` payload byte-identical on the 100k fixture (`scripts/diff_orders.py`) |

## Not in this change (verified by diff re-audit)
- `Page` → `CursorPage` rename — 122 lines, 2 exported symbols → #451
- formatter pass across `pkg/orders/**` → #452
- docstring sweep → #453
- JSON library swap proposal → #454
Each is its own review unit with its own trigger and its own reviewer.

## Risk and rollback
Blast radius as above; read path only. Rollback: revert this commit —
no schema, config, flag, or migration to unwind.
```

**Binding rules that make the card honest:**

- The exclusions block is verified against the **final** diff, not written from memory. If a hunk cannot be matched to an inclusion or an exclusion, the card is wrong and the diff is unauthorized.
- The blast radius states the command and its output count. A reassurance is not a measurement.
- The reviewer field names a person or role and their first checks. *"Please review"* is addressed to nobody, and a card addressed to nobody produces a review from nobody.
- The acceptance condition is checkable by a command or a test result. *"Looks good"* is not an accept condition; *"red on main, green on head"* is.
- Deferred items carry an identifier and an owner. An unnamed deferral is a silent deletion in a polite voice.
- Scope Ratio and Decision Count are stated when they are not obvious. A number that a reviewer can falsify in ten seconds is worth ten paragraphs of justification.

### 3.3 Architecture RFCs / ADRs — Preflighting a Decision

An RFC is a preflight for a decision that has not been made, which makes the gate load-bearing rather than defensive: without a trigger, an envelope, and a named reviewer set, an RFC degenerates into a document that argues for a preference and closes on *"thoughts?"*. The four axes map directly onto the document's skeleton — which is also why an RFC whose Context section cannot state its trigger should not be written yet.

| Preflight axis | RFC/ADR section | Fails when |
|---|---|---|
| **Trigger** | Context → *why now* | The motivation is *"the system is getting complex"* with no event, metric, or deadline |
| **Envelope** | Decision + **Non-goals** | Non-goals are omitted, so every reader assumes their own scope is included |
| **Blast Radius** | Consequences → migration surface, reversibility, cost of inaction | Costs are unestimated; reversibility is unstated; "we'll handle it" replaces a plan |
| **Reviewer** | Sign-off set: who must approve, what each role checks | *"the team"*; each reviewer checks the same thing and no one owns the risk |

```markdown
# ADR-014 — Boundary-safe paging contract for `/v2/orders`

## Preflight
- **Trigger:** INC-4477 — N+1 rows at 4 occurrences since 2026-03-01 (ticket 8812).
  Metric `orders_paging_offbyone_total` non-zero for the 7-day window ending 2026-03-28.
- **Envelope:** decide whether the boundary is expressed as an **exclusive** end bound
  in `slicePage` or as an **inclusive** one with an explicit clamp.
  **Non-goals:** renaming `Page`; adopting cursor pagination; changing the response
  envelope; the JSON library proposal (#454). None of these is decided here.
- **Blast radius:** 3 call sites (`rg -n "listOrders"`); 1 exported symbol;
  `orders.v1` wire contract; 0 schema, 0 config. Reversal cost: one commit.
- **Reviewer set:** @dana (service owner) — contract preservation;
  @sam (SRE) — reversibility and on-call implications; @priya (client team) —
  does the client tolerate the corrected row count.
- **Accept condition:** all three sign off on the chosen bound, and the decision names
  the condition that would reverse it.

## Decision
Bound is exclusive; `slicePage` clamps to `start+limit`. Retain the existing
response assembly path.

## Consequences
- Measured: `limit=50` returns 50 rows (`TestListOrdersLimit`, red on `main @ 4f9c1ab`).
- Cost of inaction: 4 support tickets; clients paginating with `offset += limit`
  silently drop rows, so the defect grows with page count, not with load.
- Reverses if: a future consumer requires inclusive bounds at the contract level
  (open question: @priya, blocking for v3 only).
```

**Rules for the RFC side of the gate:**

- **Non-goals are mandatory.** An RFC with no non-goals is an invitation for every reviewer to import their own agenda, and the review then argues about scope instead of the decision.
- The trigger must be an event with a reference. A metric, an incident, a deadline, or a contractual obligation — never a vibe about complexity.
- Blast radius is expressed as a *migration surface* with an estimate, not as an acknowledgement that work exists. Reversibility is a first-class consequence: state the operation that undoes the decision and its cost.
- Every reviewer in the sign-off set has a distinct check, or they are decorative. A reviewer whose check duplicates another's adds latency without reducing risk.
- Alternatives must be decided by a stated axis of measurement, not by rhetoric. An option rejected in a meeting and not reconstructed in the ADR will be re-litigated by every future reader.
- The **expiry clause**: name the assumption that would invalidate the decision and the reader who should notice. A decision with no expiry becomes folklore that survives its own context.

---

## 4. Verification Checklist

- [ ] **Preflight preceded generation.** The four axes — Trigger, Envelope (verb + object + acceptance criterion + exclusion list), Blast Radius, Reviewer — were written before the first token of code or documentation, and every axis that could not be answered was recorded as an `UNKNOWN` with an owner and a question rather than resolved by assumption.
- [ ] **Every hunk is authorized.** The Inverse Diff Test passes on the *final* diff: each hunk maps to an envelope clause, or to a mechanical class whose necessity was proven by a failing build or test. Zero hunks are justified by taste, "while I was in there", or "already written".
- [ ] **Blast radius is measured, not asserted.** A command was run (call-site search, exported-symbol list, contract/schema/config scan), its result count appears in the PR or ADR, and the read/write paths, deploy coupling, and reversibility are named — with a single-operation rollback stated explicitly.
- [ ] **The reviewer's cost is bounded and the boundary is declared.** Decision Count = 1 for this unit (SR ≥ 0.9 for a fix), and the artifact carries a true "Not in this change" list whose items each have an identifier and an owner — verified by re-reading the diff, not from memory.
- [ ] **The card survives a cold read.** A reader with no access to the original request can state, from the body alone, what the trigger was, what the envelope authorized, what was excluded, what the blast radius was, who accepts it, and what undoes it — without opening the diff.