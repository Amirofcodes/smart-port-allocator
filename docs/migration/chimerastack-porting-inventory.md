# ChimeraStack → Smart Port Allocator: Porting Inventory

Date: 2026-03-08  
Source repo: `../chimerastack-cli-legacy`

## Goal
Extract only the reusable port-allocation capabilities from ChimeraStack and avoid dragging template-generation coupling into the new product.

## High-value source files

| File | Current role | Reuse verdict | Notes |
|---|---|---|---|
| `src/chimera/core/port_allocator.py` | Chooses available ports from configured ranges using scanner output | **Partial reuse** | Core range parsing + first-free selection is useful. Current API is too template-oriented and has no lease/lock semantics. |
| `src/chimera/core/port_scanner.py` | Scans Docker containers for used host ports + container names | **Partial reuse** | Useful adapter logic, but currently hard-coupled to Docker SDK and no fallback when Docker unavailable. |
| `src/chimera/config/ports.yaml` | Default service ranges | **Reuse with redesign** | Good seed defaults; should become configurable profiles in new repo. |
| `tests/unit/test_port_allocation_frontend.py` | Verifies frontend template gets only needed ports | **Reference only** | Test intent useful, implementation tied to TemplateManager output `.env`. |
| `tests/unit/test_react_static_ports.py` | Verifies frontend+nginx ports and missing DB ports | **Reference only** | Good scenario coverage but not allocator-unit-level. |
| `tests/unit/test_fullstack_port_processing.py` | Fullstack create + README substitution checks | **Do not port directly** | Template rendering concern, not allocator concern. |

## Important transitive coupling discovered

1. `PortAllocator` currently depends on `PortScanner.scan()['ports']` and assumes Docker-backed scanner.
2. Allocation behavior is invoked from `TemplateManager._allocate_service_ports`, which mixes:
   - template metadata parsing,
   - compose/env generation,
   - project scaffolding concerns.
3. Current allocator is stateless between runs except instantaneous scan; no persistent lease ownership.

## Legacy behavior worth preserving

- Declarative range config loaded from YAML.
- Graceful fallback to hardcoded ranges if YAML invalid/missing.
- First available port search within an inclusive range.
- Quick “service-type + service-name” matching heuristic.

## Legacy behavior to replace

- Docker-only scanner assumption.
- No lock/lease owner model (race-prone under parallel processes).
- Weak matching heuristic (`if key in service_name.lower()`) can misclassify.
- Silent broad exception swallowing in critical config load path.

## Candidate extraction map

### Extract now (v1 foundation)
- Range model and config parsing logic (with stricter validation).
- Sequential first-free selection strategy.
- Docker scan adapter behavior.

### Rebuild around it
- Core allocator API:
  - `allocate(profile|range, owner, ttl)`
  - `release(port, owner)`
  - `list(active_only=False)`
- Lease store (file/sqlite) with lock strategy.
- Multi-scanner strategy:
  - socket bind probe (default)
  - docker host-port scan (optional enrichment)

### Defer
- Any template manager integration.
- Any compose/jinja rendering utilities.
- Chimera stack/template-specific naming logic.
