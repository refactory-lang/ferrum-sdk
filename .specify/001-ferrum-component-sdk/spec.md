# Feature Specification: Ferrum EDM Python Component SDK

**Feature Branch**: `001-ferrum-component-sdk`
**Created**: 2026-03-13
**Status**: Draft

## Overview

A Python SDK that exposes PyO3-backed bindings for the six Ferrum EDM component protocols and four core data types. Python developers write Ferrum components using familiar Python typing, the SDK validates them at development time via type stubs, and the components compile to native Rust via PyO3 and maturin.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Developer writes a Ferrum component in Python (Priority: P1)

As a data engineer, I want to implement a `MatchingStrategy` component in Python by subclassing the SDK protocol so that my IDE provides autocompletion, my code passes `mypy --strict`, and the component is usable in the Ferrum runtime without manual Rust translation.

**Why this priority**: This is the primary use case of the SDK; if developers cannot author components in Python with full type safety, the SDK has no value.

**Independent Test**: Create a minimal `MatchingStrategy` implementation, run `mypy --strict` against it, build with `maturin develop`, and invoke the component from a Rust test harness.

**Acceptance Scenarios**:

- **Scenario 1 — Protocol conformance**
  - Given a Python class implementing the `MatchingStrategy` protocol
  - When the developer runs `mypy --strict` on the file
  - Then type checking passes with zero errors

- **Scenario 2 — Runtime invocation**
  - Given a built `.so`/`.dylib` produced by `maturin develop`
  - When the Rust test harness calls `match_entities(records: Vec<EntityRecord>)` through PyO3
  - Then the Python implementation executes and returns a `Vec<MatchScore>`

- **Scenario 3 — Protocol violation detection**
  - Given a Python class that claims to implement `ValidationRule` but omits the required `validate` method
  - When the developer runs `mypy --strict`
  - Then a type error is reported identifying the missing method

---

### User Story 2 - Component translates to Rust (Priority: P1)

As a platform engineer, I want Python components written against the SDK to have a clear structural correspondence to Rust trait implementations so that the transpiler can produce idiomatic Rust from the Python source.

**Why this priority**: The SDK exists within the Refactory transpilation ecosystem; if the Python protocols do not map cleanly to Rust traits, transpilation quality degrades.

**Independent Test**: Feed a conforming Python component through the python-to-rust pipeline and verify the output implements the corresponding Rust trait.

**Acceptance Scenarios**:

- **Scenario 1 — Trait mapping**
  - Given a Python `Normaliser` implementation
  - When processed by the python-to-rust transpiler
  - Then the output contains `impl Normaliser for ...` with correct method signatures

- **Scenario 2 — Type preservation**
  - Given a Python function returning `NormalisationResult`
  - When transpiled
  - Then the Rust output uses the Ferrum `NormalisationResult` struct, not a generic type

---

### Edge Cases

- **Multiple protocol implementation**: A single class implementing both `Normaliser` and `ValidationRule` must produce valid type stubs and compile via PyO3.
- **Optional fields in EntityRecord**: Fields marked `Optional[str]` in Python must map to `Option<String>` in the Rust binding.
- **Empty MatchScore list**: Returning an empty `list[MatchScore]` from a `MatchingStrategy` must not cause a PyO3 conversion error.
- **FELValue variant types**: `FELValue` must support int, float, string, bool, and null variants and round-trip correctly between Python and Rust.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The SDK MUST expose six protocol definitions as Python `Protocol` classes: `MatchingStrategy`, `Normaliser`, `ValidationRule`, `FELFunction`, `SourceAdapter`, `SinkAdapter`.
- **FR-002**: The SDK MUST expose four data types as Python dataclasses with PyO3 bindings: `EntityRecord`, `MatchScore`, `FELValue`, `NormalisationResult`.
- **FR-003**: Each protocol MUST define all required method signatures with full type annotations.
- **FR-004**: The SDK MUST ship `.pyi` type stub files for all protocols and types so that `mypy --strict` and IDE autocompletion work without importing the compiled extension.
- **FR-005**: The SDK MUST build via `maturin build` targeting stable Rust and produce a wheel installable via `pip install`.
- **FR-006**: All four data types MUST be convertible between Python and Rust representations via PyO3 `FromPyObject` and `IntoPy` without manual serialisation.
- **FR-007**: The `FELValue` type MUST support tagged union semantics mapping to a Rust enum with variants: `Int(i64)`, `Float(f64)`, `Str(String)`, `Bool(bool)`, `Null`.
- **FR-008**: The `EntityRecord` type MUST support a flexible field map (`dict[str, FELValue]`) alongside fixed metadata fields (`id: str`, `source: str`, `timestamp: str`).
- **FR-009**: The `MatchScore` type MUST contain fields: `left_id: str`, `right_id: str`, `score: float`, `strategy: str`.
- **FR-010**: The `NormalisationResult` type MUST contain fields: `original: str`, `normalised: str`, `rule_applied: str`, `confidence: float`.

### Key Entities

- **MatchingStrategy**: Protocol requiring `match_entities(self, records: list[EntityRecord]) -> list[MatchScore]`
- **Normaliser**: Protocol requiring `normalise(self, value: str, context: dict[str, FELValue]) -> NormalisationResult`
- **ValidationRule**: Protocol requiring `validate(self, record: EntityRecord) -> tuple[bool, str]`
- **FELFunction**: Protocol requiring `evaluate(self, args: list[FELValue]) -> FELValue`
- **SourceAdapter**: Protocol requiring `read(self) -> list[EntityRecord]` and `schema(self) -> dict[str, str]`
- **SinkAdapter**: Protocol requiring `write(self, records: list[EntityRecord]) -> int` and `schema(self) -> dict[str, str]`

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: All six protocols are defined with complete type annotations and pass `mypy --strict` validation.
- **SC-002**: All four data types round-trip between Python and Rust without data loss, verified by property-based tests.
- **SC-003**: `maturin build` succeeds on Linux, macOS, and Windows CI targets with stable Rust.
- **SC-004**: A reference implementation of each protocol compiles, builds, and executes in both the Python and Rust runtimes.
- **SC-005**: Type stub files are present and enable IDE autocompletion for all public API surfaces.
- **SC-006**: Python components written against the SDK can be fed into the python-to-rust transpiler and produce compilable Rust trait implementations.
