# Requirements Checklist: Ferrum Python Component SDK

**Feature Branch**: `001-ferrum-component-sdk`
**Last Updated**: 2026-03-13

## Core Types (FR-001, FR-003 through FR-006, FR-016, FR-017)

- [ ] `EntityRecord` is importable from `ferrum_sdk.types`
- [ ] `EntityRecord` is backed by PyO3 binding to `ferrum_core::types::EntityRecord`
- [ ] `EntityRecord.get_field(name)` returns `FELValue | None`
- [ ] `EntityRecord.set_field(name, value)` sets a field value
- [ ] `EntityRecord.field_names()` returns `list[str]`
- [ ] `EntityRecord` implements `__repr__` and `__eq__`
- [ ] `MatchScore` is importable from `ferrum_sdk.types`
- [ ] `MatchScore(confidence, weight)` constructor works with valid floats
- [ ] `MatchScore` exposes read-only `confidence` and `weight` properties
- [ ] `MatchScore` raises `ValueError` when `confidence` is outside `[0.0, 1.0]`
- [ ] `MatchScore` implements `__repr__` and `__eq__`
- [ ] `FELValue` is importable from `ferrum_sdk.types`
- [ ] `FELValue` supports construction from `str`, `int`, `float`, `bool`, `None`
- [ ] `FELValue` supports construction from `list[FELValue]` and `dict[str, FELValue]`
- [ ] `FELValue` raises `TypeError` for unsupported Python types (e.g., `set`)
- [ ] `FELValue` implements `__repr__` and `__eq__`
- [ ] `NormalisationResult` is importable from `ferrum_sdk.types`
- [ ] `NormalisationResult` contains normalised `FELValue` and `changed` boolean flag
- [ ] `NormalisationResult` implements `__repr__` and `__eq__`

## Protocol Classes (FR-002, FR-007 through FR-012, FR-018)

- [ ] `MatchingStrategy` is importable from `ferrum_sdk.protocols`
- [ ] `MatchingStrategy` requires `score(self, record, candidate) -> Result[MatchScore, str]`
- [ ] `MatchingStrategy` raises `NotImplementedError` if `score` is not overridden
- [ ] `Normaliser` is importable from `ferrum_sdk.protocols`
- [ ] `Normaliser` requires `normalise(self, value) -> Result[NormalisationResult, str]`
- [ ] `Normaliser` raises `NotImplementedError` if `normalise` is not overridden
- [ ] `ValidationRule` is importable from `ferrum_sdk.protocols`
- [ ] `ValidationRule` requires `validate(self, record) -> Result[bool, str]`
- [ ] `ValidationRule` raises `NotImplementedError` if `validate` is not overridden
- [ ] `FELFunction` is importable from `ferrum_sdk.protocols`
- [ ] `FELFunction` requires `evaluate(self, args) -> Result[FELValue, str]`
- [ ] `FELFunction` exposes a `name` property returning the function's callable name
- [ ] `FELFunction` raises `NotImplementedError` if `evaluate` is not overridden
- [ ] `SourceAdapter` is importable from `ferrum_sdk.protocols`
- [ ] `SourceAdapter` requires `fetch_batch(self, batch_size) -> Result[list[EntityRecord], str]`
- [ ] `SourceAdapter` raises `NotImplementedError` if `fetch_batch` is not overridden
- [ ] `SinkAdapter` is importable from `ferrum_sdk.protocols`
- [ ] `SinkAdapter` requires `write_batch(self, records) -> Result[int, str]`
- [ ] `SinkAdapter` raises `NotImplementedError` if `write_batch` is not overridden

## Type Stubs (FR-013)

- [ ] `ferrum_sdk/__init__.pyi` stub file exists with correct exports
- [ ] `ferrum_sdk/types/__init__.pyi` stub file exists with full type annotations for all 4 types
- [ ] `ferrum_sdk/protocols/__init__.pyi` stub file exists with full type annotations for all 6 protocols
- [ ] `mypy --strict` passes on a sample component using the stubs
- [ ] `pyright` in strict mode passes on a sample component using the stubs

## API Mapping (FR-014)

- [ ] `api-map.yaml` exists in the SDK root
- [ ] Mapping for `ferrum_sdk.types.EntityRecord` -> `ferrum_core::types::EntityRecord`
- [ ] Mapping for `ferrum_sdk.types.MatchScore` -> `ferrum_core::types::MatchScore`
- [ ] Mapping for `ferrum_sdk.types.FELValue` -> `ferrum_core::types::FELValue`
- [ ] Mapping for `ferrum_sdk.types.NormalisationResult` -> `ferrum_core::types::NormalisationResult`
- [ ] Mapping for `ferrum_sdk.protocols.MatchingStrategy` -> `ferrum_core::matching::MatchingStrategy`
- [ ] Mapping for `ferrum_sdk.protocols.Normaliser` -> `ferrum_core::normalisation::Normaliser`
- [ ] Mapping for `ferrum_sdk.protocols.ValidationRule` -> `ferrum_core::validation::ValidationRule`
- [ ] Mapping for `ferrum_sdk.protocols.FELFunction` -> `ferrum_core::fel::FELFunction`
- [ ] Mapping for `ferrum_sdk.protocols.SourceAdapter` -> `ferrum_core::adapters::SourceAdapter`
- [ ] Mapping for `ferrum_sdk.protocols.SinkAdapter` -> `ferrum_core::adapters::SinkAdapter`

## Build and Distribution (FR-015)

- [ ] `Cargo.toml` is configured with PyO3 dependencies and `cdylib` crate type
- [ ] `pyproject.toml` is configured with maturin as the build backend
- [ ] `maturin develop` compiles the Rust extension and installs it locally
- [ ] `maturin build` produces a valid `.whl` file
- [ ] The built wheel installs in a clean virtual environment
- [ ] All `ferrum_sdk` types and protocols are importable after `pip install`

## Integration with python-to-rust Pipeline (SC-004)

- [ ] A sample `ISINMatcher` component translates to Rust using the API mapping
- [ ] Translated Rust code references `ferrum_core` types correctly
- [ ] Translated Rust code compiles against `ferrum_core` crate

## Round-Trip and Edge Cases (SC-007)

- [ ] `EntityRecord` round-trips through PyO3 with field equality preserved
- [ ] `MatchScore` round-trips through PyO3 with value equality preserved
- [ ] `FELValue` round-trips through PyO3 for each variant (str, int, float, bool, None, list, map)
- [ ] `NormalisationResult` round-trips through PyO3 with value equality preserved
- [ ] `EntityRecord.get_field` returns `None` for non-existent fields (no exception)
- [ ] `SourceAdapter.fetch_batch(0)` returns `Success([])`
- [ ] Python `None` maps correctly to Rust `Option::None` in `FELValue` contexts
