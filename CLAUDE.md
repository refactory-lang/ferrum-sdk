<!-- codemod-skill-discovery:begin -->
## Codemod Skill Discovery
This section is managed by `codemod` CLI.

- Core skill: `.agents/skills/codemod/SKILL.md`
- Package skills: `.agents/skills/<package-skill>/SKILL.md`
- List installed Codemod skills: `npx codemod agent list --harness antigravity --format json`

<!-- codemod-skill-discovery:end -->

## Project: ferrum-sdk

Python Component SDK for the Ferrum EDM (Entity Data Management) platform. Part of the [refactory-lang](https://github.com/refactory-lang) organization. Operators author custom pipeline components (matching strategies, normalisers, validators, FEL extensions) in constrained Python. Components are compiled to native Rust via the Refactory pipeline and run alongside the Ferrum core with zero FFI overhead.

### Architecture

- **SDK package** (`ferrum_sdk/`): Python SDK with protocols and types
  - `protocols/`: Component protocol base classes (`MatchingStrategy`, `Normaliser`, `ValidationRule`, `FELFunction`, `SourceAdapter`, `SinkAdapter`)
  - `types/`: PyO3 bindings to `ferrum_core` Rust types (`EntityRecord`, `MatchScore`, `FELValue`, `NormalisationResult`)
  - `__init__.py`: Package entry point
- **Rust bindings** (`rust/src/`): PyO3 binding source for ferrum_core types
- **Examples** (`examples/`): Example components
- **Tests** (`tests/`): Test suite
- **Specs** (`specs/`): Implementation specifications

### Running

```bash
# Build Rust bindings
cd rust && maturin develop

# Run tests
pytest

# Type check
mypy ferrum_sdk/
```

### Key Files

| File | Purpose |
|------|---------|
| `ferrum_sdk/__init__.py` | SDK package entry point |
| `ferrum_sdk/protocols/` | Component protocol base classes (6 protocols) |
| `ferrum_sdk/types/` | PyO3 bindings to ferrum_core Rust types |
| `rust/src/` | PyO3 binding implementations |
| `examples/` | Example components (matching strategies, normalisers) |

### Conventions

- All types are PyO3 bindings to actual `ferrum_core` Rust crate -- test fidelity is absolute
- API mappings (e.g., `ferrum_sdk.MatchingStrategy` -> `ferrum_core::matching::MatchingStrategy`) are maintained in this package, not in `@refactory/python-to-rust`
- Components must use `@dataclass(frozen=True)` and `returns.result.Result` for the constrained Python profile


### Speckit Workflow

This repo uses [speckit](https://github.com/speckit) for specification-driven development.

- **Specs**: `specs/<NNN-feature-name>/spec.md` — feature specifications
- **Plans**: `specs/<NNN-feature-name>/plan.md` — implementation plans with tasks
- **Checklists**: `specs/<NNN-feature-name>/checklists/` — quality gates
- **Templates**: `.specify/templates/` — spec, plan, task, checklist templates
- **Extensions**: `.specify/extensions/` — verify, sync, review, workflow hooks

**Branch convention**: Feature branches are named `<NNN>-<short-name>` matching the spec directory (e.g., `001-milestone1-pipeline`).

**Issue → Spec flow**: Issues labeled `ready-to-spec` trigger the `ready-to-spec-notify` workflow, which assigns Copilot to run the speckit workflow and produce a spec + plan + tasks.
