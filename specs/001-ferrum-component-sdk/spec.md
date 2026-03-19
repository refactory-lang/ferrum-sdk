# Feature Specification: Ferrum Python Component SDK

**Feature Branch**: `001-ferrum-component-sdk`
**Created**: 2026-03-13
**Status**: Draft
**Input**: User description: "Implement Ferrum Python Component SDK - PyO3 bindings for 6 component protocols (MatchingStrategy, Normaliser, ValidationRule, FELFunction, SourceAdapter, SinkAdapter) and 4 core types (EntityRecord, MatchScore, FELValue, NormalisationResult)"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Author a Matching Strategy Component in Python (Priority: P1)

A developer writes a custom entity matching strategy using `ferrum_sdk`. They subclass `MatchingStrategy`, implement the `score` method using `EntityRecord` and `MatchScore` types, and run `pytest` to verify correctness against real Rust-backed type instances.

**Why this priority**: Matching is the core operation in an EDM platform. If developers cannot author and test matching strategies in Python with real Rust types, the SDK delivers no value.

**Independent Test**: Create an `ISINMatcher` class that subclasses `MatchingStrategy`, calls `EntityRecord.get_field("isin")` on two records, and returns a `MatchScore`. Run `pytest` and confirm the test passes with PyO3-backed types.

**Acceptance Scenarios**:

1. **Given** a Python file importing `MatchingStrategy` from `ferrum_sdk.protocols` and `EntityRecord`, `MatchScore` from `ferrum_sdk.types`, **When** the developer subclasses `MatchingStrategy` and implements `score(self, record: EntityRecord, candidate: EntityRecord) -> Result[MatchScore, str]`, **Then** the class instantiates without error and the `score` method executes against PyO3-backed `EntityRecord` instances.
2. **Given** two `EntityRecord` instances with matching `isin` fields, **When** `score` is called, **Then** it returns `Success(MatchScore(1.0, weight))` where `MatchScore.confidence` equals `1.0`.
3. **Given** two `EntityRecord` instances with different `isin` fields, **When** `score` is called, **Then** it returns `Success(MatchScore(0.0, weight))`.

---

### User Story 2 - Author a Normaliser Component in Python (Priority: P1)

A developer writes a field normaliser that transforms raw field values into canonical forms. They subclass `Normaliser`, implement `normalise`, and return a `NormalisationResult` containing the cleaned value.

**Why this priority**: Normalisation is fundamental to data quality in EDM. It is the second most common component type after matching strategies and exercises different SDK types (`FELValue`, `NormalisationResult`).

**Independent Test**: Create an `UpperCaseNormaliser` that converts a string `FELValue` to uppercase and returns a `NormalisationResult`. Run `pytest` and verify the output value is uppercased.

**Acceptance Scenarios**:

1. **Given** a class subclassing `Normaliser` with `normalise(self, value: FELValue) -> Result[NormalisationResult, str]`, **When** called with `FELValue("hello")`, **Then** it returns `Success(NormalisationResult("HELLO"))`.
2. **Given** a `FELValue` that is `None` or empty, **When** `normalise` is called, **Then** the normaliser returns an appropriate `NormalisationResult` indicating no change or an error `Result`.

---

### User Story 3 - Author a Validation Rule in Python (Priority: P2)

A developer writes a validation rule that checks entity field constraints. They subclass `ValidationRule`, implement `validate`, and return a boolean result indicating whether the entity passes the rule.

**Why this priority**: Validation rules are simpler than matching/normalisation but essential for data integrity checks. They exercise the `EntityRecord` type in a read-only context.

**Independent Test**: Create a `RequiredFieldRule` that checks whether a given field exists and is non-empty on an `EntityRecord`. Run `pytest` and verify it returns `True` for records with the field and `False` for records without it.

**Acceptance Scenarios**:

1. **Given** a `ValidationRule` subclass implementing `validate(self, record: EntityRecord) -> Result[bool, str]`, **When** the record has a non-empty `"name"` field, **Then** it returns `Success(True)`.
2. **Given** an `EntityRecord` missing the required field, **When** `validate` is called, **Then** it returns `Success(False)`.

---

### User Story 4 - Author a Custom FEL Function (Priority: P2)

A developer extends the Ferrum Expression Language by writing a custom function. They subclass `FELFunction`, implement `evaluate`, and return a `FELValue`.

**Why this priority**: FEL extensibility is a differentiating feature of Ferrum. Custom FEL functions allow domain-specific calculations in entity pipelines.

**Independent Test**: Create a `LevenshteinDistance` FEL function that takes two `FELValue` string arguments and returns a `FELValue` integer. Run `pytest` and verify correct distance calculations.

**Acceptance Scenarios**:

1. **Given** a `FELFunction` subclass implementing `evaluate(self, args: list[FELValue]) -> Result[FELValue, str]`, **When** called with `[FELValue("kitten"), FELValue("sitting")]`, **Then** it returns `Success(FELValue(3))`.
2. **Given** an incorrect number of arguments, **When** `evaluate` is called, **Then** it returns an `Err` result with a descriptive error message.

---

### User Story 5 - Author a Source Adapter (Priority: P3)

A developer writes a data ingestion adapter that reads entities from an external source. They subclass `SourceAdapter`, implement `fetch_batch`, and yield `EntityRecord` instances.

**Why this priority**: Source adapters are less common (most users rely on built-in adapters), but the protocol must exist for custom integrations. They exercise iterator/batch patterns over `EntityRecord`.

**Independent Test**: Create a `CSVSourceAdapter` that reads from an in-memory CSV string and yields `EntityRecord` objects. Verify that field values on the yielded records match the CSV data.

**Acceptance Scenarios**:

1. **Given** a `SourceAdapter` subclass implementing `fetch_batch(self, batch_size: int) -> Result[list[EntityRecord], str]`, **When** called with `batch_size=10`, **Then** it returns a list of up to 10 `EntityRecord` instances.
2. **Given** the source has no more records, **When** `fetch_batch` is called, **Then** it returns `Success([])` (empty list).

---

### User Story 6 - Author a Sink Adapter (Priority: P3)

A developer writes a data output adapter that writes processed entities to an external destination. They subclass `SinkAdapter`, implement `write_batch`, and accept `EntityRecord` instances.

**Why this priority**: Sink adapters mirror source adapters for data output. Like source adapters, they are less commonly customised but must be available.

**Independent Test**: Create an `InMemorySinkAdapter` that collects written `EntityRecord` instances in a list. Verify that after calling `write_batch`, the internal list contains the expected records.

**Acceptance Scenarios**:

1. **Given** a `SinkAdapter` subclass implementing `write_batch(self, records: list[EntityRecord]) -> Result[int, str]`, **When** called with a list of 5 `EntityRecord` instances, **Then** it returns `Success(5)` indicating all records were written.
2. **Given** an empty list of records, **When** `write_batch` is called, **Then** it returns `Success(0)`.

---

### User Story 7 - Python Component Translates to Rust via python-to-rust Pipeline (Priority: P1)

A developer authors a component using the SDK, validates it with the profile validator, and submits it to the Refactory python-to-rust transformation pipeline. The pipeline uses the SDK's API mapping to convert `ferrum_sdk` types/protocols to their `ferrum_core` Rust equivalents.

**Why this priority**: Translation to Rust is the entire reason the SDK exists. Without this, components would only work in Python. The API mapping is critical for the pipeline to know how to convert SDK types.

**Independent Test**: Take a complete `ISINMatcher` Python component, run it through the python-to-rust pipeline with the SDK's `api-map.yaml`, and verify the output Rust code references `ferrum_core::matching::MatchingStrategy` and `ferrum_core::types::EntityRecord`.

**Acceptance Scenarios**:

1. **Given** a Python component using `from ferrum_sdk.protocols import MatchingStrategy` and `from ferrum_sdk.types import EntityRecord, MatchScore`, **When** the python-to-rust pipeline processes it with the SDK's API mapping, **Then** the output Rust code uses `ferrum_core::matching::MatchingStrategy`, `ferrum_core::types::EntityRecord`, and `ferrum_core::types::MatchScore`.
2. **Given** a component using `@dataclass(frozen=True)` and `returns.result.Result`, **When** translated, **Then** the Rust output uses a `struct` with `derive` attributes and `Result<T, E>` return types.

---

### User Story 8 - IDE Type Checking with Stub Files (Priority: P2)

A developer opens their component in VS Code or PyCharm and gets full autocomplete, type hints, and error highlighting for all `ferrum_sdk` types and protocols without needing to compile the Rust extension.

**Why this priority**: Developer experience is critical for adoption. Without type stubs, developers cannot discover the API or catch type errors before running tests.

**Independent Test**: Open a Python file importing `ferrum_sdk` types in VS Code with Pylance. Verify that `EntityRecord.get_field` shows the correct signature, `MatchScore` constructor shows `(confidence: float, weight: float)`, and incorrect argument types are flagged as errors.

**Acceptance Scenarios**:

1. **Given** `.pyi` stub files are installed alongside the SDK, **When** a developer imports `EntityRecord` and calls `record.get_field("name")`, **Then** the IDE shows the return type as `FELValue | None` and offers autocomplete for `get_field`.
2. **Given** a developer writes `MatchScore("not_a_float", 1.0)`, **When** the IDE type checker runs, **Then** it reports a type error indicating `str` is not assignable to `float`.

---

### Edge Cases

- What happens when `EntityRecord.get_field` is called with a field name that does not exist? The method must return `None` (not raise an exception), matching Rust's `Option::None`.
- What happens when a `FELValue` is constructed with an unsupported Python type (e.g., a `set`)? The PyO3 binding must raise a `TypeError` at construction time with a clear message listing supported types.
- What happens when a protocol method returns a plain value instead of a `Result`? The SDK must raise a `TypeError` explaining that protocol methods must return `Result[T, str]`.
- How does the SDK handle Python `None` vs Rust `Option::None`? `None` in Python must map to `Option::None` in Rust for all `FELValue` contexts.
- What happens when `SourceAdapter.fetch_batch` is called with `batch_size=0`? It must return `Success([])` without error.
- What happens when `SinkAdapter.write_batch` receives records with incompatible schemas? It must return an `Err` result describing the schema mismatch.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The SDK MUST expose 4 core types as PyO3 bindings to `ferrum_core` Rust crate types: `EntityRecord`, `MatchScore`, `FELValue`, and `NormalisationResult`, importable from `ferrum_sdk.types`.
- **FR-002**: The SDK MUST expose 6 protocol base classes: `MatchingStrategy`, `Normaliser`, `ValidationRule`, `FELFunction`, `SourceAdapter`, and `SinkAdapter`, importable from `ferrum_sdk.protocols`.
- **FR-003**: `EntityRecord` MUST provide `get_field(name: str) -> FELValue | None` to retrieve field values, `set_field(name: str, value: FELValue) -> None` to set field values, and `field_names() -> list[str]` to list all field names.
- **FR-004**: `MatchScore` MUST be constructible as `MatchScore(confidence: float, weight: float)` where `confidence` is in `[0.0, 1.0]` and `weight` is a positive float. It MUST expose `confidence` and `weight` as read-only properties.
- **FR-005**: `FELValue` MUST support construction from Python types `str`, `int`, `float`, `bool`, `None`, `list[FELValue]`, and `dict[str, FELValue]`, mirroring Rust's `FELValue` enum variants.
- **FR-006**: `NormalisationResult` MUST contain the normalised `FELValue` and a boolean `changed` flag indicating whether the value was modified.
- **FR-007**: `MatchingStrategy` protocol MUST require subclasses to implement `score(self, record: EntityRecord, candidate: EntityRecord) -> Result[MatchScore, str]`.
- **FR-008**: `Normaliser` protocol MUST require subclasses to implement `normalise(self, value: FELValue) -> Result[NormalisationResult, str]`.
- **FR-009**: `ValidationRule` protocol MUST require subclasses to implement `validate(self, record: EntityRecord) -> Result[bool, str]`.
- **FR-010**: `FELFunction` protocol MUST require subclasses to implement `evaluate(self, args: list[FELValue]) -> Result[FELValue, str]` and expose a `name` property returning the function's FEL-callable name.
- **FR-011**: `SourceAdapter` protocol MUST require subclasses to implement `fetch_batch(self, batch_size: int) -> Result[list[EntityRecord], str]`.
- **FR-012**: `SinkAdapter` protocol MUST require subclasses to implement `write_batch(self, records: list[EntityRecord]) -> Result[int, str]`.
- **FR-013**: The SDK MUST provide `.pyi` type stub files for `ferrum_sdk`, `ferrum_sdk.types`, and `ferrum_sdk.protocols` so that IDEs and type checkers (mypy, pyright) can validate component code without compiling the Rust extension.
- **FR-014**: The SDK MUST include an `api-map.yaml` file mapping each `ferrum_sdk` type and protocol to its corresponding `ferrum_core` Rust path, consumed by the python-to-rust transformation pipeline.
- **FR-015**: The SDK MUST be buildable via `maturin develop` (for local development) and `maturin build` (for distribution), producing a wheel that includes the compiled PyO3 extension module `ferrum_sdk._native`.
- **FR-016**: All PyO3 bindings MUST implement Python `__repr__` and `__eq__` methods for debuggability and test assertions.
- **FR-017**: `MatchScore` MUST reject `confidence` values outside `[0.0, 1.0]` by raising `ValueError` at construction time.
- **FR-018**: Protocol base classes MUST raise `NotImplementedError` with a descriptive message if a required method is not overridden by the subclass.

### Key Entities

- **EntityRecord**: The fundamental data unit in Ferrum. Represents a single entity (e.g., a financial instrument, a counterparty) as a collection of named fields. Each field holds a `FELValue`. Maps to `ferrum_core::types::EntityRecord` in Rust.
- **MatchScore**: The result of comparing two `EntityRecord` instances. Contains a `confidence` float (0.0 to 1.0) and a `weight` float for weighted aggregation. Maps to `ferrum_core::types::MatchScore`.
- **FELValue**: A dynamically-typed value used throughout the Ferrum Expression Language. Variants: String, Integer, Float, Boolean, Null, List, Map. Maps to `ferrum_core::types::FELValue`.
- **NormalisationResult**: The output of a normaliser, pairing the normalised `FELValue` with a `changed` flag. Maps to `ferrum_core::types::NormalisationResult`.
- **Protocol Classes**: Abstract base classes defining component contracts. Each protocol maps to a Rust trait in `ferrum_core` (e.g., `MatchingStrategy` -> `ferrum_core::matching::MatchingStrategy`).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: All 4 core types (`EntityRecord`, `MatchScore`, `FELValue`, `NormalisationResult`) are importable from `ferrum_sdk.types` and backed by PyO3 bindings that construct, serialize, and compare identically to their Rust counterparts.
- **SC-002**: All 6 protocol classes (`MatchingStrategy`, `Normaliser`, `ValidationRule`, `FELFunction`, `SourceAdapter`, `SinkAdapter`) are importable from `ferrum_sdk.protocols` and enforce their required method signatures at instantiation time.
- **SC-003**: A developer can write a complete component (e.g., `ISINMatcher`) using only `ferrum_sdk` imports, run `mypy --strict` on it with zero errors, and execute it via `pytest` with all tests passing.
- **SC-004**: The `api-map.yaml` file contains correct mappings for all 10 SDK types/protocols, and the python-to-rust pipeline successfully translates a sample component referencing each mapping.
- **SC-005**: `maturin build` produces a valid wheel for the current platform, and `pip install` of that wheel makes all `ferrum_sdk` types and protocols available in a fresh Python environment.
- **SC-006**: Type stub files enable IDE autocomplete for all public methods and constructors of the 4 core types, verified by running `pyright` in strict mode on a sample component with zero errors.
- **SC-007**: PyO3 bindings for all 4 core types pass round-trip equality tests: constructing a value in Python, passing it to Rust, and reading it back produces an equal Python value.
