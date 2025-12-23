# Plan
## Objective
Investigate the mechanisms underlying list-processing tasks in LLMs to understand how they encode and execute filtering operations that mirror the generic 'filter' function of functional programming.

## Hypothesis
1. A small number of attention heads (filter heads) encode a compact representation of the filtering predicate in their query states at certain tokens.
2. The predicate representation in filter heads is general and portable: it can be extracted and reapplied to execute the same filtering operation on different collections, formats, languages, or tasks.
3. Transformer LMs can exploit a different strategy for filtering: eagerly evaluating if an item satisfies the predicate and storing this intermediate result as a flag directly in the item representations.

## Methodology
1. Apply causal mediation analysis using activation patching on diverse list-processing tasks. Identify filter heads by patching query states (qsrc) from a source prompt to a destination prompt with different predicates and collections, measuring logit changes of target items.
2. Learn a sparse binary mask over all attention heads using Distributed Causal Mediation (DCM) to identify heads that cause the score of the target item to increase when their query states are patched.
3. Test generalization across linguistic variations (different formats, languages, question placement), information types (object categorization, profession, nationality), and different filter-reduce tasks (SelectOne, SelectFirst, SelectLast, Counting, CheckPresence).
4. Perform ablation studies by modifying attention patterns of filter heads to measure their necessity for task performance. Compare filter heads with other specialized head types (Function Vector, Concept Induction heads).
5. Investigate dual filtering strategies by comparing question-before vs question-after formats, testing for is_match flags stored in item representations through flag ablation and swapping experiments.

## Experiments
### Within-task portability: Information types and linguistic variations
- What varied: Semantic domain (object type, profession, nationality, landmark location, rhyme), collection size (2-7 distractors), presentation format (single line, bulleted), language (English, Spanish, French, Hindi, Thai), question placement (before/after items)
- Metric: Causality score (whether LM predicts correct target item after patching), ΔLogit (logit difference)
- Main result: Filter heads maintain high causality (0.836-0.863) across object types and professions, moderate (0.504-0.576) for nationality/landmark, near-zero (0.041) for rhyme. High cross-lingual transfer (0.775-0.951). Question-after shows 0.863 causality; question-before drops to 0.020.

### Cross-task portability
- What varied: Task type: SelectOne, SelectOne-MCQ, SelectFirst, SelectLast, Counting, CheckPresence. Tested both head transfer and predicate (qsrc) transfer.
- Metric: Causality score for both head evaluation across tasks and predicate transfer across tasks
- Main result: SelectOne/SelectFirst/SelectLast show ≥70% cross-causality. Counting heads show asymmetric pattern (Select* heads fail on Counting, but Counting heads partially work on Select* tasks). CheckPresence shows poor within-task causality (0.09).

### Ablation study: Necessity of filter heads
- What varied: Ablated filter heads vs random heads by modifying attention patterns. Tested on all six tasks.
- Metric: LM accuracy after ablation (baseline 100% on test set)
- Main result: Ablating filter heads (<2% of total heads) drops accuracy dramatically for Select* tasks (SelectOne: 22.5%, SelectOne-MCQ: 0.4%, SelectFirst: 13.1%, SelectLast: 9.22%). Minimal effect on Counting (89.80%) and CheckPresence (98.61%). Random ablation: 97-100%.

### Key states carry item semantics
- What varied: Two-part intervention: (1) patch qsrc for filter head, (2) swap key states between target item (ctarg) and unrelated item (cother)
- Metric: Causality score (whether LM selects cother after combined intervention), ΔLogit
- Main result: Causality score of 0.783 (432/552 examples) with ΔLogit = 8.26 ± 3.35 for SelectOne object categorization, confirming key states encode semantic properties that predicates evaluate.

### Dual filtering strategy: Question-before vs question-after
- What varied: Question placement (before/after items). Tested is_match flag through ablation (removing flag) and swapping (transferring flag between items).
- Metric: Accuracy after flag ablation, logit changes after flag swapping across layers
- Main result: Question-before: ablating is_match drops accuracy to 46.09% (vs 96.06% for question-after). Flag swapping causes LM to select cflag in early layers for question-before, minimal effect for question-after.

### Training-free probe for concept detection
- What varied: Layer selection for filter head [35,19] in Llama-70B. Tested classification using qcls · WK^ℓj h for 238 objects across 16 classes.
- Metric: Probe accuracy compared to logit lens baseline
- Main result: Filter head probe achieves 0.81 ± 0.02 accuracy at optimal layers, comparable to logit lens baseline, demonstrating zero-shot concept detection without training.