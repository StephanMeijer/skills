# Ledger Template

Create `LEDGER.md` during setup. Update it at the end of every round, including rounds that crashed or stopped on a guard. Keep each attempt row to one line; details live in the attempt's `RESULT.md`.

```markdown
# Champion challenger ledger: <task>

Contract: v<n> (CONTRACT.md)

## Champion
Attempt <NNN>-<slug>: <one-line approach>
Validation <metric> <score> ± <noise>; confirmation <score>
Components: <representation, model, data treatment, objective, framing>
Path: attempts/<NNN>-<slug>/

## Attempts
| # | Idea | Axis | Validation | Δ vs champion [95 % CI] | Decision | Salvaged |
|---|---|---|---|---|---|---|
| 000 | majority class | baseline | ... | — | baseline | — |

## Idea map
| Axis | Tried | Outcome |
|---|---|---|
| Representation | raw; TF-IDF | TF-IDF won (003) |
| Model family | logistic; GBDT | |
| Data | | |
| Objective | | |
| Framing | | |
| Composition | | |

## Salvage pile
- [S1] from 004: <finding> — evidence: <counts or slice metric beyond noise> — status: open | absorbed by <NNN> | discarded

## Falsified hypotheses
- 005: <hypothesis> — falsified by <result>

## Streak and budget
Rounds without a win: <n> / plateau limit <n>
Budget used: <rounds or hours> / <total>

## Open questions to the user
- <question>; do not act on it until answered
```

## Attempt `RESULT.md`

```markdown
# Attempt <NNN>-<slug>

<hypothesis block from references/radical-ideas.md>

## Result
Validation <metric>: <score>; champion <score>; Δ <mean> [95 % CI <lo>, <hi>]
Confirmation: <score or "not run">
Decision: win | loss | tie | crash | budget overrun
Runtime: <time>; hardware: <hardware>

## Error comparison with champion
Challenger right, champion wrong: <n>; champion right, challenger wrong: <n>
Pattern in the challenger's wins: <class, slice, or none found>
Per-class change beyond noise: <list>

## Salvage
- <finding with evidence, or "none">

## Reproduce
<exact command>
```
