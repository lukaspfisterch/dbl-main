# Changelog

## [0.3.0] - 2025-12-26

- Breaking: refocused dbl-main as deterministic projection over dbl-core events
- Added projection contract and state model with PROVEN phase
- Renamed RunnerStatus.COMPLETED to RunnerStatus.FINALIZED
- Removed policy pipelines and config loaders from public surface

## [0.1.0] - 2025-12-04

Initial version.

### Core

- Pipeline: policy orchestration
- PolicyRegistry, PipelineRegistry
- ContextBuilder

### Policies

- RateLimitPolicy
- ContentSafetyPolicy
- Policy base class

### Audit

- AuditLogger

