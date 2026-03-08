# Test Migration Map (ChimeraStack → Smart Port Allocator)

Date: 2026-03-08

## Purpose
Map legacy test intent into new allocator-focused tests.

## Legacy tests reviewed

| Legacy test | Intent | Port to SPA? | New target test |
|---|---|---|---|
| `test_port_allocation_frontend.py` | Frontend stack should allocate frontend ports only | **Intent yes, file no** | `tests/core/test_profile_allocation.py::test_frontend_profile_allocates_expected_keys` |
| `test_react_static_ports.py` | Frontend + web ports present; DB/admin absent | **Intent yes** | `tests/core/test_profile_allocation.py::test_profile_excludes_unrequested_groups` |
| `test_fullstack_port_processing.py` | Fullstack project create end-to-end | **No direct port** | Replace with focused allocator API tests, not scaffolding tests |
| `test_template_manager.py::test_create_project_port_allocation` | Consecutive creates should avoid DB port collision | **Yes (core)** | `tests/core/test_allocator_sequential.py::test_allocations_do_not_reuse_active_ports` |

## New test suites to create

### 1) Core allocator behavior
- `tests/core/test_config_parsing.py`
  - valid YAML range parsing
  - invalid format rejection
  - fallback/default profile behavior
- `tests/core/test_allocator_sequential.py`
  - first-free in range
  - exhausted range -> explicit error
  - deterministic ordering

### 2) Lease & ownership
- `tests/lease/test_lease_lifecycle.py`
  - allocate with owner + ttl
  - release by same owner
  - reject release by different owner
  - lease expiry reclamation

### 3) Concurrency/locking
- `tests/lease/test_locking.py`
  - dual-process allocation on same range
  - no duplicate assignment under contention

### 4) Scanner adapters
- `tests/adapters/test_socket_scanner.py`
  - detects occupied port via bind probe
- `tests/adapters/test_docker_scanner.py` (optional/skippable)
  - mocked docker client parsing host bindings
  - robust against empty/malformed bindings

### 5) CLI contract
- `tests/cli/test_allocate_command.py`
- `tests/cli/test_release_command.py`
- `tests/cli/test_list_command.py`
- `tests/cli/test_doctor_command.py`

## What not to carry over
- Template generation assertions (`docker-compose.yml`, `.env` file shape, README rendering).
- Chimera stack variant matrix tests.
- Any tests requiring full scaffolding output.

## Definition of done for migration test baseline
- 100% pass on core + lease + cli suites without requiring Docker daemon.
- Docker adapter tests isolated and optional (mocked or feature-gated).
- At least one contention test proving no duplicate allocations under concurrency.
