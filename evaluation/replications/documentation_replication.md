# Documentation: Replication of "LLMs Process Lists With General Filter Heads"

## Goal

Replicate the core experiment from the paper investigating how Large Language Models (LLMs) perform filtering operations over lists of items using specialized attention heads called "filter heads." The key hypothesis is that a small number of attention heads encode a compact representation of the filtering predicate in their query states, and this representation can be transferred between different contexts.

## Data

### Source Data
- **Dataset**: SelectOne task from the `data_save/selection/objects.json` file
- **Categories**: 16 object categories including fruit, vehicle, furniture, animal, etc.
- **Format**: Each category contains a list of objects (e.g., fruits: Apple, Banana, Orange, etc.)
- **Prompt Templates**: 4 templates available for constructing SelectOne prompts

### Sample Structure
Each sample consists of:
- A list of options (1 target + N distractors)
- A prompt asking to identify an object of a specific category
- The target object (correct answer)

### Counterfactual Pairs
For the patching experiment:
- **Source sample**: Question about category A (e.g., "Which is a fruit?")
- **Destination sample**: Question about category B (e.g., "Which is a vehicle?") with a hidden category A item

## Method

### 1. Query State Caching
For each attention head in the model:
- Hook the QKV projection layer (c_attn in GPT-2, q_proj in Llama)
- Cache query projections at specified token positions (typically last 3 tokens)
- Store projections indexed by (layer, head, token_position)

### 2. Query State Patching
Apply the patching intervention:
- Take cached query states from source prompt
- Replace corresponding query states in destination prompt
- Map token positions (e.g., -1 -> -1, -2 -> -2, -3 -> -3)

### 3. Effect Measurement
Measure the patching effect:
- Compare logits before and after patching
- Track logit changes for:
  - Destination object (expected answer for unpatched prompt)
  - Track object (hidden source-category item)
- Compute delta logits (patched - baseline)

### 4. Head Identification
Identify filter heads:
- Test each head individually
- Rank heads by effect on track object logit
- Heads with strong positive effects are candidate filter heads

## Results

### Model Used
- GPT-2 (124M parameters) due to computational constraints
- Original paper used Llama-3.3-70B-Instruct

### Head Analysis
Top 5 heads by effect on track object logit:
1. Layer 11, Head 4: Δtrack = 0.6875
2. Layer 10, Head 7: Δtrack = 0.4375
3. Layer 9, Head 4: Δtrack = 0.3750
4. Layer 1, Head 7: Δtrack = 0.3125
5. Layer 7, Head 1: Δtrack = 0.3125

### Multi-Sample Experiment (N=10)
- Mean Δ Track Object: 0.90 ± 1.43
- Mean Δ Dest Object: 1.41 ± 1.22
- Positive effect on track object: 60% of samples

### Comparison to Original Paper
| Metric | Original (Llama-70B) | Replication (GPT-2) |
|--------|---------------------|---------------------|
| Causality Score | 0.836-0.863 | N/A (model doesn't solve task) |
| ΔLogit | 8.26 ± 3.35 | 0.90 ± 1.43 |
| Task Accuracy | ~100% | <20% |

## Analysis

### What Was Replicated
1. **Methodology**: The core patching mechanism works correctly
2. **Head-Specific Effects**: Some heads show stronger effects than others
3. **Effect Direction**: Patching tends to increase track object logits

### What Was Not Replicated
1. **Effect Magnitude**: Much smaller effects with GPT-2
2. **Task Performance**: GPT-2 cannot reliably solve the SelectOne task
3. **Filter Head Localization**: Cannot identify clear "filter heads" in GPT-2

### Reasons for Differences
1. **Model Size**: GPT-2 (124M) vs Llama-3.3-70B (70B parameters)
2. **Training**: GPT-2 not trained for instruction following
3. **Architecture**: Different attention mechanisms and head counts

### Artifacts Generated
- `replication.ipynb`: Jupyter notebook with full implementation
- `head_effects_heatmap.png`: Visualization of head-wise patching effects
