# Feature Specification: Ferrum EDM Python Component SDK

**Feature Branch**: `001-ferrum-component-sdk`
**Created**: 2026-03-13
**Status**: Draft

## Overview

The Ferrum SDK provides a Python package that defines the six component protocols and four core data types used by the Ferrum Entity Data Management platform. Developers author components in Python against these protocols, benefit from full type-checking via stubs, and the components compile to native Rust extensions through PyO3 and maturin. The same component code runs in both the Python development runtime and the compiled Rust production runtime.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Developer writes a Ferrum component in Python using the SDK (Priority: P1)

As a data engineer, I want to implement a `MatchingStrategy` protocol in Python so that I can define entity matching logic using familiar Python syntax and have it type-checked by mypy/pyright before deployment.

**Why this priority**: Authoring components is the primary use case for the SDK; every other feature depends on having well-defined protocols.

**Independent Test**: Write a minimal `MatchingStrategy` implementation, run mypy against it, and confirm zero type errors.

**Acceptance Scenarios**:

```
Scenario 1: Implement MatchingStrategy
  Given a Python class that implements the MatchingStrategy protocol
  When the class defines the required match(entity_a: EntityRecord, entity_b: EntityRecord) -> MatchScore method
  Then mypy reports zero errors on the file

Scenario 2: Implement Normaliser
  Given a Python class that implements the Normaliser protocol
  When the class defines the required normalise(value: str) -> NormalisationResult method
  Then mypy reports zero errors on the file

Scenario 3: Implement ValidationRule
  Given a Python class that implements the ValidationRule protocol
  When the class defines the required validate(record: EntityRecord) -> list[str] method
  Then mypy reports zero errors on the file

Scenario 4: Implement FELFunction
  Given a Python class that implements the FELFunction protocol
  When the class defines the required evaluate(*args: FELValue) -> FELValue method
  Then mypy reports zero errors on the file

Scenario 5: Implement SourceAdapter
  Given a Python class that implements the SourceAdapter protocol
  When the class defines the required read() -> Iterator[EntityRecord] method
  Then mypy reports zero errors on the file

Scenario 6: Implement SinkAdapter
  Given a Python class that implements the SinkAdapter protocol
  When the class defines the required write(records: list[EntityRecord]) -> int method
  Then mypy reports zero errors on the file
```

---

### User Story 2 - Component translates to Rust via PyO3 (Priority: P1)

As a platform operator, I want Python components to compile into native Rust extensions via maturin so that production workloads execute at near-native speed without requiring a Python interpreter.

**Why this priority**: The dual-runtime promise is the core differentiator of Ferrum; if PyO3 compilation fails the SDK has no production path.

**Independent Test**: Run `maturin build` on a sample component and verify the resulting `.so`/`.dylib` loads in both Python and Rust test harnesses.

**Acceptance Scenarios**:

```
Scenario 1: Build with maturin
  Given a valid MatchingStrategy implementation
  When I run maturin build --release
  Then the build succeeds with exit code 0 and produces a wheel artifact

Scenario 2: Load compiled component in Python
  Given the wheel from Scenario 1
  When I pip install the wheel and import the component
  Then the component's match() method is callable and returns a MatchScore

Scenario 3: Load compiled component in Rust
  Given the compiled .so artifact
  When a Rust test harness loads it via PyO3
  Then calling match() through the FFI boundary returns the expected MatchScore
```

---

### Edge Cases

- **Protocol with optional methods**: If a protocol defines optional lifecycle hooks (e.g., `setup()`, `teardown()`), omitting them must not cause type errors or runtime failures.
- **EntityRecord with missing fields**: Accessing a non-existent field on `EntityRecord` must raise a clear `KeyError` in Python and return `None`/`Option::None` in Rust.
- **FELValue type coercion**: Passing an `int` where a `float` FELValue is expected must auto-coerce in Python but produce a compile-time error in Rust.
- **Large record batches**: `SinkAdapter.write()` receiving 1M+ records must not cause unbounded memory growth — the SDK must support or document streaming semantics.
- **Thread safety**: PyO3-compiled components must be safe to call from multiple Rust threads (`Send + Sync`).

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: SDK MUST define six Python protocols: `MatchingStrategy`, `Normaliser`, `ValidationRule`, `FELFunction`, `SourceAdapter`, `SinkAdapter`.
- **FR-002**: SDK MUST define four data types: `EntityRecord`, `MatchScore`, `FELValue`, `NormalisationResult`.
- **FR-003**: `EntityRecord` MUST behave as a dictionary-like object with string keys and support field access by name.
- **FR-004**: `MatchScore` MUST contain at minimum a `score: float` field (0.0-1.0) and a `matched_fields: list[str]` field.
- **FR-005**: `FELValue` MUST be a union type supporting `str`, `int`, `float`, `bool`, `list[FELValue]`, and `None`.
- **FR-006**: `NormalisationResult` MUST contain the `original: str`, `normalised: str`, and `confidence: float` fields.
- **FR-007**: SDK MUST ship `.pyi` type stub files for all protocols and types, compatible with mypy and pyright.
- **FR-008**: SDK MUST compile via `maturin build` using PyO3 bindings without manual Rust boilerplate from the component author.
- **FR-009**: Each protocol MUST define a clear set of required methods with fully typed signatures.
- **FR-010**: SDK MUST provide a `@ferrum_component` decorator or registration mechanism that marks a class for PyO3 export.
- **FR-011**: All six protocols MUST have Rust trait equivalents auto-generated or hand-maintained in the SDK crate.

### Key Entities

| Entity | Description |
|---|---|
| `MatchingStrategy` | Protocol for entity pair comparison, returns `MatchScore` |
| `Normaliser` | Protocol for string/value normalisation, returns `NormalisationResult` |
| `ValidationRule` | Protocol for record validation, returns list of error strings |
| `FELFunction` | Protocol for Ferrum Expression Language custom functions |
| `SourceAdapter` | Protocol for reading entity records from external systems |
| `SinkAdapter` | Protocol for writing entity records to external systems |
| `EntityRecord` | Dictionary-like record representing a single entity |
| `MatchScore` | Numeric score plus metadata from a matching operation |
| `FELValue` | Union type for expression language values |
| `NormalisationResult` | Result container for normalisation operations |

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: All six protocols are importable from `ferrum_sdk` and pass mypy `--strict` checks.
- **SC-002**: All four data types are importable, constructible, and pass mypy `--strict` checks.
- **SC-003**: A sample component implementing each of the six protocols compiles via `maturin build` with zero errors.
- **SC-004**: Compiled components load and execute correctly in both a Python test harness and a Rust test harness.
- **SC-005**: Type stub coverage is 100% — every public class, method, and attribute has a `.pyi` entry.
- **SC-006**: CI pipeline includes mypy, maturin build, and cross-runtime integration tests for every PR.
