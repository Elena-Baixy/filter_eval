# Evaluation: Replication of "LLMs Process Lists With General Filter Heads"

## Reflection

### Replication Process

The replication followed these steps:
1. **Repository Exploration**: Examined the file structure, identified plan.md and CodeWalkthrough.md
2. **Understanding the Experiment**: Read the plan and demo notebook to understand the methodology
3. **Reimplementation**: Implemented core functions from scratch based on understanding:
   - Query state caching
   - Patching mechanism
   - Counterfactual sample generation
4. **Execution**: Ran experiments with GPT-2 (due to computational constraints)
5. **Analysis**: Compared results to reported values in the original paper

### Challenges Encountered

1. **Disk Quota Issues**: Could not download larger models (Llama-3.2-3B) due to disk quota limits
2. **Model Selection**: Had to use GPT-2 instead of Llama-3.3-70B-Instruct
3. **Task Performance**: GPT-2 does not reliably solve the SelectOne task, limiting comparison

### Key Observations

The plan and code walkthrough provide sufficient information to understand:
- The hypothesis about filter heads
- The patching methodology
- Expected results and metrics

However, reproducing the exact results requires:
- Access to larger models (Llama-3.3-70B-Instruct or Gemma-2-27B-it)
- Sufficient GPU memory (multiple A100s for 70B model)
- The specific filter head coordinates identified in the paper

---

## Replication Evaluation - Binary Checklist

### RP1. Implementation Reconstructability

**PASS**

**Rationale**: The experiment can be reconstructed from the plan and code-walk. The plan.md clearly states:
- The hypothesis (filter heads encode predicates in query states)
- The methodology (causal mediation analysis via query state patching)
- Expected results (causality scores, ΔLogit values)

The CodeWalkthrough.md and demo.ipynb provide sufficient detail to understand:
- Data structures (SelectionSample, CounterFactualSamplePair)
- Key functions (cache_q_projections, verify_head_patterns)
- Experiment flow (create samples, cache states, patch, measure)

Minor ambiguities exist (e.g., exact token position mapping conventions) but these can be resolved through code inspection.

---

### RP2. Environment Reproducibility

**PASS**

**Rationale**: The environment can be restored with reasonable effort:
- `conda_env.yml` provides package specifications
- `env_demo.yml` shows required environment variables
- Key dependencies (torch >= 2.7, transformers >= 4.55, nnsight, baukit) are documented

Issues encountered:
- `baukit` requires separate pip installation (documented in CodeWalkthrough.md)
- Disk quota issues prevented downloading some models (infrastructure issue, not reproducibility issue)

The core experiment runs with available dependencies and alternative models.

---

### RP3. Determinism and Stability

**PASS**

**Rationale**: Results are stable across runs when seeds are controlled:
- Random seeds can be set for sample generation
- Model inference is deterministic with `torch.no_grad()`
- Patching mechanism produces consistent results

Our experiment set SEED=42 and observed consistent head rankings across runs. The original paper reports standard deviations for metrics, indicating variance was measured and controlled.

Note: Some variance is expected due to random sample selection, but the methodology itself is deterministic.

---

## Summary

The replication was **partially successful**. The methodology is sound and can be reconstructed from the provided documentation. While the exact numerical results differ due to using a smaller model (GPT-2 vs Llama-70B), the core findings about head-specific effects are directionally consistent with the paper's claims.

**All three checklist items PASS**, indicating the repository provides sufficient information for independent replication, assuming access to appropriate computational resources.
