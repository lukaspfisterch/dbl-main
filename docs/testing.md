# Testing and Guarantees

DBL Main enforces its design guarantees through executable tests. This document maps guarantees to test files.

## Guarantees

| Guarantee | Description |
|-----------|-------------|
| Context Immutability | `BoundaryContext` is never mutated by policies or pipelines |
| Deterministic Aggregation | Same input produces same output, decisions accumulate predictably |
| Error Isolation | Exceptions in policies propagate cleanly, context remains unchanged |
| Decision Transparency | All policy decisions are observable via `PolicyDecision` and `BoundaryResult` |

## Test Structure

```
tests/
├── test_maxims.py         # Core design maxims (immutability, aggregation, determinism)
├── test_determinism.py    # Determinism under load and parallel execution
├── test_error_handling.py # Exception propagation, edge cases
├── test_edge_cases.py     # Boundary values, unusual inputs
├── test_integration.py    # End-to-end flows, registry, audit
├── test_fuzz.py           # Property-based tests (requires hypothesis)
├── test_pipeline.py       # Pipeline behavior
├── test_policies.py       # Individual policy tests
├── test_registry.py       # Registry behavior
├── test_config.py         # Configuration loading
└── test_audit.py          # Audit logger
```

## Guarantee to Test Mapping

### Context Immutability

Tested in:
- `test_maxims.py::test_maxim3_no_mutation`
- `test_maxims.py::test_maxim3_effective_metadata_is_independent`
- `test_maxims.py::test_maxim1_policies_see_original_context`
- `test_error_handling.py::test_context_unchanged_after_exception`
- `test_fuzz.py::test_invariant_context_unchanged`

Key assertions:
- `context.metadata` is never modified
- `effective_metadata` is a deep copy, not an alias
- Policies always see the original context

### Deterministic Aggregation

Tested in:
- `test_maxims.py::test_maxim4_deterministic_under_load`
- `test_maxims.py::test_maxim4_deterministic_parallel`
- `test_maxims.py::test_maxim2_modifications_accumulate`
- `test_determinism.py::test_pipeline_deterministic_under_repeated_calls`
- `test_fuzz.py::test_invariant_deterministic`

Key assertions:
- 1000+ repeated calls produce identical results
- Parallel execution produces consistent results
- Modifications accumulate in defined order

### Error Isolation

Tested in:
- `test_error_handling.py::test_exception_in_policy_propagates`
- `test_error_handling.py::test_context_unchanged_after_exception`
- `test_error_handling.py::test_block_before_exception_stops_pipeline`
- `test_edge_cases.py::test_block_then_modify_never_reached`

Key assertions:
- Exceptions propagate, not swallowed silently
- Context unchanged even after exception
- Block stops pipeline before exception-throwing policies

### Decision Transparency

Tested in:
- `test_maxims.py::test_pipeline_decisions_order_preserved`
- `test_integration.py::test_audit_logger_captures_result`
- `test_pipeline.py::test_pipeline_stops_on_block`

Key assertions:
- All decisions accessible via `result.decisions`
- Decision order matches policy order
- `BoundaryResult.describe()` provides serializable snapshot

## Running Tests

### Quick run (no optional dependencies)

```bash
pip install -e .[test]
pytest
```

### With property tests (hypothesis)

```bash
pip install -e .[test-fuzz]
pytest
```

### Selective runs

```bash
# Fast: skip stress and fuzz tests
pytest -m "not stress and not fuzz"

# Stress tests only
pytest -m stress

# Integration tests only
pytest -m integration

# Fuzz tests only (requires hypothesis)
pytest -m fuzz
```

## Test Markers

| Marker | Description |
|--------|-------------|
| `@pytest.mark.stress` | Long-running or high-load tests |
| `@pytest.mark.fuzz` | Hypothesis-based property tests |
| `@pytest.mark.integration` | End-to-end integration tests |

## CI Strategy

### Core CI (every commit)

- Python 3.11, 3.12
- `pip install .[test]`
- `pytest -m "not fuzz"`

### Deep CI (nightly or pre-release)

- `pip install .[test-fuzz]`
- `pytest` (all tests including fuzz)

## Adding Tests

When adding guarantees or behavior:

1. Identify which guarantee category applies
2. Add test to the appropriate file
3. Use markers if stress/fuzz/integration
4. Update this document if adding new guarantee category

