# Evaluation Contract

Write `CONTRACT.md` during setup, with the user, before any model is trained. After setup, no round may change it. A change needs the user's approval and starts a new contract version; scores from different versions are not comparable, so rescore the champion under the new version.

## Splits

- Split once, save the example indices or IDs under `splits/`, and record their SHA-256 hashes.
- Group before splitting: every example from the same entity (user, patient, document, session, near-duplicate cluster) goes into one split. Deduplicate exact and near duplicates across splits.
- Stratify by class when the classes are imbalanced. Split by time when the model will predict the future; never let training data come from after validation data.
- Keep three parts:
  - **train**: anything goes, including internal cross-validation;
  - **validation**: scored every round and used to pick the champion;
  - **test**: locked. Scored once, when the user asks for the final report.
- Define the **confirmation view** used before promoting a winner: k-fold cross-validation over train plus validation with fixed folds, or a separate confirmation split. Small datasets favor folds; large ones favor a split.

## Metric

- Pick one selection metric that matches the cost of mistakes: macro-F1 or balanced accuracy for imbalanced classes where each class matters, log-loss or Brier score when probabilities are consumed, AUC-PR for rare positives, accuracy only for balanced classes with equal error costs.
- Record secondary metrics for the report (per-class precision and recall, confusion matrix, calibration), but select only on the primary one.
- Record hard constraints (latency, size, interpretability) as pass or fail gates applied before the metric.

## Scoring Script

- Write `evaluate.*` once. It takes a predictions file with one row per validation example (example ID, predicted class, class probabilities when the metric needs them) and prints the metric. It reads labels itself; attempts never read validation labels for training.
- It must reject a predictions file with missing, extra, or duplicate IDs.
- Record its SHA-256 hash. Each round verifies it before scoring.

## Win Rule

A challenger wins only when it beats the champion by more than noise on the same validation examples:

1. Measure noise in round 0 by retraining the baseline with at least three seeds; record the standard deviation of the metric.
2. Compute a paired bootstrap over validation examples: resample example indices with replacement (at least 1,000 times, fixed seed), score both prediction sets on each resample, and take the difference.
3. The challenger wins when the lower bound of the 95 % interval of the difference is above zero **and** the mean difference exceeds the minimum meaningful gain recorded here. For losses where lower is better, flip the sign.
4. Before promotion, the challenger must also beat the champion on the confirmation view. When it wins on validation but not there, record it as a loss with a note "validation-only win" and salvage it.

## Budget and Stop Limits

- Per-round budget: wall time, hardware, memory.
- Total budget: rounds or hours.
- Plateau limit: the number of consecutive rounds without a win after which the loop reports and suggests stopping.
- Target score on validation and confirmation.

## Template

```markdown
# Evaluation contract v1 — <task>

Approved by the user on <date>.

- Data: <path>, target `<column>`, unit of example: <row/document/...>
- Grouping key: <column or rule>; dedup rule: <rule>
- Splits: train <n> / validation <n> / test <n> (locked); confirmation view: <5-fold over train+validation | split of n>
- Split hashes: train <sha256>, validation <sha256>, test <sha256>, folds <sha256>
- Selection metric: <metric> (higher|lower is better); secondary: <list>
- Gates: <latency, size, interpretability, or none>
- Scoring script: evaluate.<ext>, sha256 <hash>
- Win rule: paired bootstrap, 1,000 resamples, seed <n>; minimum meaningful gain <delta>
- Baseline noise (std over <k> seeds): <value>
- Budget: <per round>; total <rounds or hours>; plateau limit <n> rounds
- Target: <score> on validation and confirmation
- Allowed: packages <list or "ask">; pretrained models <yes/no, which>; downloads <yes/no>; external data <yes/no>; paid APIs <yes/no, limit>
```
