# Documentation: Filter Heads Replication

## Goal

Replicate the core findings from "LLMs Process Lists With General Filter Heads" paper, which investigates how LLMs perform filtering operations over lists using specialized attention heads called "filter heads."

## Data

### Source
- **Repository**: `/net/scratch2/smallyan/filter_eval`
- **Data files**: `data_save/selection/objects.json` containing 16 object categories (fruit, vehicle, furniture, etc.) with 14-15 items each
- **Task**: SelectOne - identifying a specific category item from a list of mixed options

### Generated Samples
- Source prompts: "Which object from the following list is a fruit?" with mixed options
- Destination prompts: "Which object from the following list is a vehicle?" with mixed options including a tracked fruit item

## Method

### 1. Model Selection
- Used **Llama-3-8B-Instruct** (smallest available model) instead of the paper's 70B/27B models
- The paper provides predefined filter heads only for 70B and 27B models, so we performed our own filter head localization for 8B

### 2. Filter Head Localization
We implemented the causal mediation analysis from scratch:
1. Created source and destination prompts with different filtering predicates
2. Captured query states from source prompts using forward hooks
3. Patched query states from source to destination at the last 3 token positions
4. Measured the change in logits for target items (fruit vs vehicle)

### 3. Causality Score Calculation
- **Fruit Δlogit**: Change in logit for the tracked fruit item after patching
- **Vehicle Δlogit**: Change in logit for the correct vehicle item after patching
- **Causality Score**: Fruit Δlogit - Vehicle Δlogit (higher = more predicate transfer)

## Results

### Identified Filter Heads (Llama-3-8B-Instruct)
| Layer | Head | Causality Score |
|-------|------|-----------------|
| 17 | 24 | 4.44 |
| 13 | 18 | 2.00 |
| 27 | 5 | 1.25 |
| 27 | 20 | 1.19 |
| 24 | 27 | 1.00 |
| 13 | 1 | 0.88 |

### Predicate Transfer Effects (10 sample pairs)
- **Average Fruit Δlogit**: +6.997 (increases toward fruit prediction)
- **Average Vehicle Δlogit**: -2.838 (decreases away from vehicle prediction)
- **Effect consistency**: 10/10 samples showed positive fruit delta and negative vehicle delta

### Comparison with Paper
| Metric | Paper (70B) | Replication (8B) |
|--------|-------------|------------------|
| Filter head layers | 28-50 | 13-27 |
| Predicate transfer | Yes | Yes |
| Causality scores | 0.836-0.863 | Variable per head |

## Analysis

### Successful Replications
1. ✓ Filter heads exist in smaller models (8B) with similar qualitative behavior
2. ✓ Query state patching successfully transfers predicates between contexts
3. ✓ Effects are consistent across multiple test samples
4. ✓ Filter heads are concentrated in middle-to-later layers (proportionally similar to 70B)

### Limitations
1. Used 8B model instead of 70B/27B due to efficiency - absolute metrics may differ
2. Did not replicate cross-task or cross-lingual experiments
3. Did not replicate ablation studies or dual filtering strategy experiments
4. Binary causality rate (prediction flip) was 0% because baseline vehicle predictions were very strong

### Notes on Implementation
- Original code uses `nnsight` library which had compatibility issues with current environment
- Reimplemented core functionality using PyTorch forward hooks
- Results are qualitatively consistent with paper despite implementation differences
