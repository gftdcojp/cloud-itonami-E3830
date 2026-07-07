# ADR-0001: Recovery Advisor ⊣ Traceability Governor architecture

## Status

Accepted. `cloud-itonami-isic-3830` promoted from `:blueprint` to
`:implemented` in the `kotoba-lang/industry` registry.

## Context

`cloud-itonami-isic-3830` publishes an OSS business blueprint for
materials recovery: community recycling, reuse, repair, material
traceability and local circular-economy operations, run by a
certified operator so a community keeps its own custody and impact
records instead of renting a closed SaaS. Like every prior actor in
this fleet, the blueprint alone is not an implementation: this ADR
records the governed-actor architecture that promotes it to real,
tested code, following the same langgraph-clj StateGraph + independent
Governor + Phase 0→3 rollout pattern established by `cloud-itonami-
isic-6511` (life insurance) and applied across forty-three prior
siblings, most recently `cloud-itonami-isic-3030` (aerospace
manufacturing).

## Decision

### Decision 1: this fleet's second circular-economy/infrastructure vertical

Following `3600` (water-safety operations) and `6190` (telecom
access), and immediately after `3030` (this fleet's first
manufacturing vertical), `cloud-itonami-isic-3830` is this fleet's
SECOND vertical grounded in physical circular-economy/resource-
recovery infrastructure -- distinct from `3600`'s/`6190`'s utility-
service domains and from `3030`'s manufacturing domain. The
distinguishing concern here is TRACEABILITY of a real-world material's
chain of custody and grade, not a utility-service delivery guarantee
or a manufacturing-QA guarantee.

### Decision 2: entity and op shape

The primary entity is a `batch` (a material batch collected/recovered
for grading and resale/reuse, analogous to `aerospace.store`'s
`assembly` or `telecom.store`'s `line`). Five ops: `:batch/intake`
(directory upsert, no capital risk), `:grading/verify` (per-
jurisdiction materials-recovery grading evidence checklist, never auto
-- analogous to `water.operation`'s `:jurisdiction/assess`),
`:contamination/screen` (contamination-flag screening, unconditional-
evaluation discipline, never auto), `:actuation/certify-material-
grade` (POSITIVE, high-stakes -- certifying the batch's real material
grade ahead of sale/reuse), and `:actuation/publish-impact-report`
(POSITIVE, high-stakes -- publishing the real impact/ESG report
referencing the batch's own weighed and verified records). This is the
SAME dual-actuation-on-one-entity shape `school`/`association`/
`leasing`/`behavioral`/`secondary`/`card`/`water`/`telecom`/
`aerospace` all use -- and, like `aerospace`, BOTH actuations here are
POSITIVE, matching the majority shape in this fleet's history.

### Decision 3: `contamination-percentage-exceeds-maximum?` -- the 4th MAXIMUM-ceiling check

Following `facility.registry/occupancy-exceeds-capacity?` (1st),
`school.registry/class-size-exceeds-maximum?` (2nd) and `card.
registry/settlement-amount-exceeds-authorized?` (3rd) -- confirmed via
grep before writing this docstring, avoiding the false-precedent-claim
risk `leasing`'s ADR-0001 documents -- `recovery.registry/
contamination-percentage-exceeds-maximum?` applies the SAME single-
value-vs-ceiling-comparison shape to a batch's own measured
contamination percentage against its own recorded maximum-allowed-
for-grade threshold. Unlike the two-sided range check family
(`testlab`/`conservation`/`water`/`aerospace`), a contamination
percentage naturally floors at 0 and only has a MEANINGFUL upper
bound -- a single-sided ceiling is the correct shape here, not a
two-sided range, and this distinction is deliberate (see Alternatives
considered). It gates only `:actuation/certify-material-grade` --
directly grounded in this blueprint's own published Offer/Trust
Control framing that grade certification is the act requiring
contamination-ceiling sufficiency.

### Decision 4: `contamination-flag-unresolved-violations` -- the 28th unconditional-evaluation screening grounding

Following the discipline `casualty.governor/sanctions-violations`
established and twenty-seven prior siblings (most recently `aerospace.
governor/ndt-defect-unresolved-violations`, the 27th) have applied,
`contamination-flag-unresolved-violations` is evaluated
UNCONDITIONALLY -- not scoped to a specific op -- so `:contamination/
screen` itself can HARD-hold on its own finding, not merely gate the
downstream actuation. This is the 28th distinct grounding of this
exact discipline, and the FIRST specifically for a contamination-flag
concept -- deliberately modeled as a SEPARATE boolean concept from the
numeric contamination-percentage ceiling check (Decision 3), the same
"two independent concepts on the same underlying real-world
phenomenon" shape `water.governor` uses (`contaminant-level-out-of-
range?` vs. `threshold-breach-unresolved?`, both about water
contamination but independently modeled). Exercised in tests/demo via
`:contamination/screen` DIRECTLY against an already-flagged batch, not
via an actuation op against an unscreened batch -- the "screen the
screening op directly, not the actuation op" lesson `parksafety`'s
ADR-2607071922 Decision 5 established, now applied for an EIGHTEENTH
consecutive sibling (`facility`=8th, `school`=9th, `association`=10th,
`leasing`=11th, `behavioral`=12th, `secondary`=13th, `card`=14th,
`water`=15th, `telecom`=16th, `aerospace`=17th, `recovery`=18th).

### Decision 5: dedicated double-actuation-guard booleans

`:material-grade-certified?`/`:impact-report-published?` are dedicated
booleans on the `batch` record, never a single `:status` value -- the
same discipline every prior sibling governor's guards establish,
informed by `cloud-itonami-isic-6492`'s real status-lifecycle bug
(ADR-2607071320).

### Decision 6: Store protocol, MemStore + DatomicStore parity

`recovery.store/Store` is implemented by both `MemStore` (atom-backed,
default for dev/tests/demo) and `DatomicStore` (`langchain.db`-backed),
proven to satisfy the same contract in `test/recovery/
store_contract_test.clj` -- the same seam every sibling actor uses so
swapping the SSoT backend is a configuration change, not a rewrite.

### Decision 7: Phase 0→3 rollout

Phase 3's `:auto` set has exactly one member, `:batch/intake` (no
capital risk). `:grading/verify` and `:contamination/screen` are
never auto-eligible at any phase (matching every sibling's screening-
op posture), and `:actuation/certify-material-grade`/`:actuation/
publish-impact-report` are permanently excluded from every phase's
`:auto` set -- a structural fact, not a rollout milestone, enforced by
BOTH `recovery.phase` and `recovery.governor`'s `high-stakes` set
independently.

### Decision 8: no bespoke domain capability lib

This vertical's batch records are practice-specific rather than a
shared cross-operator data contract, so `recovery.*` runs on the
generic robotics/telemetry/forms/dmn/bpmn/audit-ledger/optimization
stack only -- the same posture `9412`/`8720`/`8521`/`3030` and others
without a bespoke capability lib already establish.

### Decision 9: mock + LLM advisor pair

`recovery.recoveryadvisor` provides `mock-advisor` (deterministic,
default everywhere -- the actor graph and governor contract run
offline) and `llm-advisor` (backed by `langchain.model/ChatModel`,
with a defensive EDN-proposal parser so a malformed LLM response
degrades to a safe low-confidence noop rather than ever auto-
certifying a material grade or auto-publishing an impact report).

### Decision 10: blueprint.edn field-sync fixes

Two stale-scaffold inconsistencies in `blueprint.edn`, discovered
during the standard "survey blueprint scaffold" step before writing
any code, were fixed as part of this promotion (the same class of fix
`card.6619`'s, `water.3600`'s, `telecom.6190`'s and `aerospace.3030`'s
own ADR-0001s document):

1. `:itonami.blueprint/id` was the stale pre-rename value
   `"cloud-itonami-3830"` (missing `isic-`), while the repo folder,
   README title and this actor's own `:business-id` already use the
   corrected `cloud-itonami-isic-3830`. Fixed to match.
2. `:itonami.blueprint/required-technologies`/`:optional-technologies`
   were missing entirely despite the `kotoba-lang/industry` registry's
   own entry for `"3830"` already stating `[:robotics :forms
   :telemetry :optimization :audit-ledger :bpmn :dmn]` / `[:identity]`.
   Fixed to match the registry exactly.

## Alternatives considered

- **Modeling `contamination-percentage-exceeds-maximum?` as a two-
  sided range check** (matching `testlab`/`conservation`/`water`/
  `aerospace`'s established family). Rejected: a contamination
  percentage has no meaningful lower bound distinct from 0 -- there is
  no failure mode analogous to "too little contamination" the way a
  water contaminant level or an assembly's dimensional tolerance can
  fail on EITHER side. Forcing a two-sided shape here would add a
  meaningless `:contamination-min-allowed` field with no real-world
  referent; the honest, minimal shape is a single-sided MAXIMUM-
  ceiling check, joining `facility`/`school`/`card`'s family instead.
- **Collapsing `contamination-percentage-exceeds-maximum?` and
  `contamination-flag-unresolved?` into a single check.** Rejected to
  match `water.governor`'s precedent of keeping a numeric ground-truth
  recompute and an independently-screened boolean flag as TWO separate
  concepts on the same underlying phenomenon (contamination) -- a
  batch could have low, in-spec contamination-percentage YET still
  carry an unresolved contamination FLAG from a physical inspection
  finding (e.g. a foreign-object flag not captured by the percentage
  metric alone), so collapsing them would lose a real distinction.
- **A single actuation (certification only), treating impact-report
  publication as a lower-stakes administrative note.** Rejected: the
  blueprint's own Trust Controls explicitly state "impact claims
  reference weighed or verified records" as an independent invariant
  from grade-certification correctness itself -- collapsing it into a
  non-high-stakes op would contradict the blueprint's own stated
  posture that BOTH the certification and the public claim are
  independently gated.

## Consequences

- Forty-fourth actor in this fleet (43 implemented before this build),
  and the SECOND circular-economy/infrastructure vertical.
- Confirms the MAXIMUM-ceiling check family generalizes to a fourth,
  genuinely distinct domain (contamination-grade QA), following
  `facility`/`school`/`card`.
- Establishes the 28th unconditional-evaluation screening grounding,
  the first for a contamination-flag concept, and demonstrates the
  "two independent concepts on one phenomenon" shape `water.governor`
  established generalizes to a new domain.
- Two pre-existing `blueprint.edn` inconsistencies (stale ID, missing
  required/optional-technologies fields) fixed as in-scope minor
  consistency work, consistent with how `card.6619`/`water.3600`/
  `telecom.6190`/`aerospace.3030` handled the same class of issue.
