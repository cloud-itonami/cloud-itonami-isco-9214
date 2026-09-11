# cloud-itonami-isco-9214

Open Occupation Blueprint for **ISCO-08 9214**: Garden and Horticultural
Labourers.

This repository designs a forkable OSS business for a garden and
horticultural-site scheduling and logistics coordination practice: a site
scheduling and supply-coordination robot manages crew/task records under a
governor-gated actor, so a garden/horticultural-site crew keeps its own
operating records instead of renting a closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/gardenhort/` implements the
`GardenHorticulturalActor` as a `langgraph.graph/state-graph`
(`gardenhort.actor`) wired to a `Garden and Horticultural Labourer Advisor`
(`gardenhort.advisor`) and an independent `GardenHorticulturalGovernor`
(`gardenhort.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok?) +-> :request-approval (:escalate?, human-in-the-loop interrupt) +->
:hold (:hard?)`. 24 tests / 52 assertions green (`kbb -M:test`). HARD
invariants (always hold, never overridable): worker provenance, site
provenance, no-actuation (`:effect` must be `:propose`), a closed
op-allowlist (`:log-work-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — nothing else may
ever be proposed), and a permanent, unconditional block on any proposal
that would directly finalize a garden-work-execution decision (e.g.
finalizing the garden-maintenance operation) *or* a site-safety-clearance
decision (e.g. declaring the site safety cleared), or that would override
a site safety supervisor's judgment. Always-escalate paths (human sign-off
regardless of confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)): `:flag-safety-concern`
(always) and `:coordinate-supply-order` above the registered cost
threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot
performs the physical domain work**. Here a site scheduling/logistics
coordination robot performs crew scheduling, planting/maintenance-log/
progress-record logging and plants/soil/tool-supplies procurement
coordination for a garden/horticultural-site crew, under an actor that
proposes actions and an independent **GardenHorticulturalGovernor** that
gates them. The governor never dispatches hardware itself, never performs
garden work on the site itself, and never finalizes a garden-work-execution
decision or a site-safety-clearance decision, and never overrides a site
safety supervisor's judgment; `:high`/`:safety-critical` actions (such as a
flagged power-tool-hazard/pesticide-exposure/weather-exposure concern, or
an above-threshold supply order) require human sign-off. **This actor
coordinates SITE SCHEDULING/LOGISTICS ONLY — it never performs garden work
itself and never makes a site-safety-clearance decision itself.**

## Core Contract

```text
worker roster + site registration + safety-reporting policy
        |
        v
Garden and Horticultural Labourer Advisor -> GardenHorticulturalGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize a garden-work-execution decision, finalize a site-safety-
clearance decision (e.g. declaring the site safety cleared), override a
site safety supervisor's judgment, suppress an operating record, or
disclose sensitive data without governor approval and audit evidence.

## Hazard scope

Garden and horticultural labourers perform manual outdoor garden/
horticulture work. This actor's safety-concern channel exists because of
a stacked, independent hazard scope:

- power-tool hazard (mowers, trimmers, hedge cutters)
- pesticide/fertilizer chemical exposure
- outdoor weather exposure

Every flagged concern in any of these categories always escalates to
human sign-off — never auto-committed, regardless of confidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `9214`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
