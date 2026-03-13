# ferrum-sdk

Python Component SDK for the Ferrum EDM platform. Operators author custom pipeline components (matching strategies, normalisers, validators, FEL extensions) in constrained Python. Components are compiled to native Rust via the Refactory pipeline and run alongside the Ferrum core with zero FFI overhead.

## How It Works

1. **Author** — Write component logic in Python using `ferrum_sdk` types and protocols.
2. **Test** — Run `pytest` against real Rust implementations via PyO3 bindings + shadow libraries.
3. **Validate** — Profile validator ensures code is translatable (no exceptions, no dynamic types).
4. **Compile** — Refactory pipeline translates to Rust and compiles alongside Ferrum core.

## Component Protocols

| Protocol | Purpose | Python Base |
|----------|---------|-------------|
| `MatchingStrategy` | Score entity similarity | `ferrum_sdk.protocols.MatchingStrategy` |
| `Normaliser` | Transform field values | `ferrum_sdk.protocols.Normaliser` |
| `ValidationRule` | Validate entity fields | `ferrum_sdk.protocols.ValidationRule` |
| `FELFunction` | Custom FEL expression functions | `ferrum_sdk.protocols.FELFunction` |
| `SourceAdapter` | Data ingestion logic | `ferrum_sdk.protocols.SourceAdapter` |
| `SinkAdapter` | Data output logic | `ferrum_sdk.protocols.SinkAdapter` |

## Types

All types are PyO3 bindings to the actual `ferrum_core` Rust crate — test fidelity is absolute.

| Type | Purpose |
|------|---------|
| `EntityRecord` | Core entity representation |
| `MatchScore` | Match result with confidence |
| `FELValue` | FEL expression value |
| `NormalisationResult` | Normalisation output |

## Example

```python
from ferrum_sdk.protocols import MatchingStrategy
from ferrum_sdk.types import EntityRecord, MatchScore
from dataclasses import dataclass
from returns.result import Result, Success

@dataclass(frozen=True)
class ISINMatcher(MatchingStrategy):
    weight: float = 1.0

    def score(self, record: EntityRecord, candidate: EntityRecord) -> Result[MatchScore, str]:
        r_isin = record.get_field("isin")
        c_isin = candidate.get_field("isin")
        if r_isin == c_isin:
            return Success(MatchScore(1.0, self.weight))
        return Success(MatchScore(0.0, self.weight))
```

## API Mapping

ferrum_sdk type mappings are maintained in this package, not in `@refactory/python-to-rust`'s api-map.yaml:

```yaml
ferrum_sdk.MatchingStrategy: ferrum_core::matching::MatchingStrategy
ferrum_sdk.types.EntityRecord: ferrum_core::types::EntityRecord
ferrum_sdk.types.MatchScore: ferrum_core::types::MatchScore
# ... etc
```

## License

Apache-2.0
