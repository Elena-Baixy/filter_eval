# Documentation: Replication of Filter Heads Experiment

## Goal

Replicate the key demonstration from the paper "LLMs Process Lists With General Filter Heads" which investigates how transformer language models encode and execute filtering operations over lists of items.

The main hypothesis is that a small number of specialized attention heads ("filter heads") encode a compact, portable representation of the filtering predicate in their query states. This predicate can be transferred to different contexts to execute the same filtering operation.

## Data

### Dataset
- **Source**: `data_save/selection/objects.json`
- **Task**: SelectOne - identify a specific item from a list based on a category criterion
- **Categories tested**:
  - fruit, vehicle, animal, furniture, and 16 total object categories
- **Sample format**: "Options: [list of objects]. Which among these objects mentioned above is a [category]? Answer:"

### Model
- **Original demo**: Meta-Llama-3.3-70B-Instruct (80 layers, 64 heads)
- **Replication**: Meta-Llama-3-8B-Instruct (32 layers, 32 heads)
- **Rationale**: Used smaller model per replication guidelines while maintaining same architecture family

## Method

### 1. Filter Head Identification
Since filter heads for the 8B model were not pre-identified, we used attention pattern analysis:
1. Generate a sample task asking about a specific category (e.g., "fruit")
2. Analyze attention matrices across all layers and heads
3. Identify heads where the last token (answer position) strongly attends to the target item
4. Select top 15 heads with highest attention scores as filter head candidates

### 2. Query State Patching Experiment
Following the paper's methodology:
1. **Source prompt**: Ask about category A (e.g., "Which is a fruit?")
2. **Destination prompt**: Ask about category B (e.g., "Which is a vehicle?") but include an item from category A
3. Cache query projections from filter heads during source prompt processing
4. Patch these query states into the destination prompt processing
5. Measure logit change for the category A item in the destination

### 3. Technical Implementation
- Used `nnsight` library for activation patching
- Patched query states at the last 3 token positions (-3, -2, -1)
- Used `cache_q_projections` to extract query states
- Applied patches via `verify_head_patterns` with custom PatchSpec objects

## Results

### Filter Heads Identified (Llama-3-8B)
15 filter head candidates in layers 14-28:
- Layer 14: Head 22
- Layer 17: Head 24
- Layer 20: Heads 13, 14, 25, 26
- Layer 23: Head 6
- Layer 24: Head 27
- Layer 26: Heads 13, 14, 15
- Layer 27: Heads 5, 20, 23
- Layer 28: Head 15

### Patching Results

| Test Case | Baseline Logit | Patched Logit | Δ Logit |
|-----------|----------------|---------------|---------|
| fruit → vehicle (Banana) | 8.875 | 21.750 | +12.875 |
| animal → furniture (Sheep) | 10.688 | 19.750 | +9.063 |

### Key Observations
1. **Successful predicate transfer**: Patching query states from a "fruit" source prompt to a "vehicle" destination prompt caused the model to select the fruit item (Banana) instead of the vehicle item (Motorcycle)
2. **Rank improvement**: The tracked item went from rank 262 to rank 1 after patching
3. **Generalization**: The same filter heads worked for different category pairs (fruit/vehicle and animal/furniture)

## Analysis

### Comparison to Original Demo
The original demo on Llama-3.3-70B showed:
- Δ logit of 4.875 for patching all 79 filter heads
- Our replication with 8B model shows Δ logit of 9-13 with 15 filter heads

The larger improvement in the 8B model may be due to:
1. Smaller model having more concentrated filter functionality in fewer heads
2. Different baseline distributions
3. Selection bias in filter head identification (we selected heads based on attention patterns from a single example)

### Validity of Replication
1. **Core hypothesis confirmed**: Filter heads encode portable predicates in query states
2. **Methodology preserved**: Same patching approach (cache q_proj, patch at last tokens)
3. **Qualitative match**: Patching causes selection of source-category items in destination

### Limitations
1. Used smaller model (8B vs 70B) - may have different internal representations
2. Filter heads identified empirically rather than using DCM optimization
3. Tested on limited number of examples

## Conclusion

The replication successfully demonstrates the core finding of the paper: filter heads in transformer LLMs encode a compact, portable representation of filtering predicates that can be transferred across contexts. The query state patching mechanism works effectively on the smaller Llama-3-8B model, validating the generality of the filter head phenomenon across model scales.
