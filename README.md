# cloud-itonami-3830

Open Business Blueprint for **ISIC Rev.5 3830**: materials recovery.

This repository designs a forkable OSS business for community recycling,
reuse, repair, material traceability and local circular-economy operations.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a sorting and handling robot performs grading, baling and loadout of recovered materials under an actor that proposes
actions and an independent **Traceability Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
operating near heavy bales, baling equipment or conveyors) require human sign-off.

## Core Contract

```text
collection records + material scans + buyer specs
        |
        v
Recovery Advisor -> Traceability Governor -> route, hold, or approve
        |
        v
chain-of-custody ledger + impact report
```

The advisor can suggest routes, buyers and reuse actions, but cannot falsify
material grade, chain of custody or impact claims.

## Runbook

- Start with collection and sorting records.
- Add material-grade recommendations.
- Add buyer matching and logistics.
- Publish traceable impact reports.

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

Code and implementation templates are AGPL-3.0-or-later.
