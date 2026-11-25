# Pre-registered test (plan v3)

This folder holds the plan for a pre-registered test of the OPEN engine on play after 2026-11-24, and the record of when it was locked. The test has not been scored yet.

| File | What it holds |
| --- | --- |
| `analysis-plan-v3.md` | The plan, published exactly as locked |
| `analysis-plan-v3-lock.md` | The lock record: hashes, dates and counts only, no player data |

## Checking the plan

The plan was locked on 2026-11-24 at 19:55 UTC. Its sha256 is:

```
04aca3b314633bb63c6fc4df27c2aa8029249789b3b9dd246f119510b0d0b3b3
```

It is the `plan_sha256` in the lock record. To check the file here, run `sha256sum analysis-plan-v3.md`. The plan is published byte for byte, so nothing in it has been edited for publication.

## Reading it

- The plan's status line still says "DRAFT. Not locked." Editing it at the lock would have changed the plan's sha256. The lock record is the proof that the lock happened.
- Links to issues and pull requests (`#26`, `#40` and so on) point to the evaluation's private repository, so readers cannot open them. The plan states each decision in its own text.
- Commands such as `v3-lock` and `v3-score` belong to the evaluation code, which is not published.
- Owner decision 11 records that the plan runs no comparison with any outside rating system.

## What happens next

New play is counted each week. The test stops at 400 matches eligible after burn-in, or on 2027-06-30, whichever comes first. It is then scored once, and that first scored run is the result. Any later run, or any change to the plan, the code or the engine pin, is reported as a deviation (plan section 13).
