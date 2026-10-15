# Radical Ideas

A round's idea is **radical** when it changes the approach along at least one axis to a value the ledger has not tried, and the change rests on a hypothesis about the data rather than on tuning. These do not count as radical on their own: changing hyperparameters, seeds, or the number of epochs; swapping one gradient-boosting library for another; adding a layer.

Before picking, read the idea map in `LEDGER.md`. Prefer the axis that is least explored and most promising given the error analysis and the salvage pile. When every obvious value is explored, combine two axes or reframe the problem.

## Axes

**Representation.** Raw features; hand-engineered domain features; aggregates over the grouping entity; text as bag of words or character n-grams; embeddings from a pretrained encoder; learned representations fine-tuned end to end; image or audio features from a pretrained backbone; graph features from relationships between examples.

**Model family.** Linear and logistic models; k-nearest neighbors on a good representation; tree ensembles and gradient boosting; neural networks suited to the modality (MLP, CNN, transformer); fine-tuned foundation models; zero-shot or few-shot classification with a language model when the contract allows it; rule-based systems mined from the data.

**Data.** Clean or relabel noisy examples found by confident learning or disagreement between models; remove duplicates and contradictions; augment the minority classes; generate synthetic examples; use unlabeled data through pseudo-labeling or self-training; use external data when allowed; re-weight examples by difficulty or recency.

**Objective.** Class weights or cost-sensitive loss; focal loss; label smoothing; optimizing the metric directly or through a surrogate; per-class decision thresholds tuned on train folds; calibration before thresholding.

**Problem framing.** Hierarchical classification (coarse class first, then fine); one-versus-rest or pairwise specialists for the confused classes; a dedicated model for a hard slice routed by a gate; anomaly detection for a rare class; metric learning and nearest-prototype classification; ordinal formulation for ordered classes.

**Composition.** Ensembles of diverse champions and salvaged losers; stacking with a meta-model on out-of-fold predictions; mixture of experts per slice; distilling a large model into a small one that meets the gates.

## Deriving the Next Idea

Base the idea on evidence, not on a list walk:

- Read the champion's confusion matrix and its worst slices. An idea that targets the largest block of errors beats a generic one.
- Read the salvage pile. A losing attempt that won one slice suggests a specialist, a feature, or an ensemble member.
- Read the falsified hypotheses. Do not retry a falsified idea unless a salvage item changes the premise; say what changed.
- When the champion plateaus within one family, jump to a different family or framing rather than refining.

## Writing the Hypothesis

Record in `RESULT.md` before training:

```markdown
Idea: <one line>
Axis: <axis: old value -> new value>
Hypothesis: <what structure in the data this exploits that the champion misses>
Expected effect: <which classes or slices should improve>
Falsified if: <result that would show the hypothesis is wrong>
Built on: champion <attempt> + salvage <items>
```
