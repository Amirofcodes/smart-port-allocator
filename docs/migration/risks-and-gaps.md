# Risks and Gaps for Smart Port Allocator v1

Date: 2026-03-08

## Top risks

1. **Concurrency collisions**
   - Legacy allocator has no persistent lock/lease mechanism.
   - Parallel CLI/agent calls can allocate the same port.
   - Mitigation: file lock + atomic lease write before returning allocation.

2. **Docker hard dependency**
   - Legacy scanner uses Docker SDK directly (`docker.from_env()`), brittle when Docker absent or socket inaccessible.
   - Mitigation: default scanner = socket bind probe; Docker scanner as optional adapter.

3. **Weak range mapping heuristic**
   - Legacy `if key in service_name.lower()` can mis-route services.
   - Mitigation: explicit profiles and named allocation groups (`frontend`, `web`, `db`, `admin`) with strict schema.

4. **Silent failure modes**
   - Broad exception catching in config load may hide bad config.
   - Mitigation: fail-fast validation with explicit errors and CLI `doctor` diagnostics.

5. **Lease orphaning**
   - If a process crashes, leased ports may remain blocked forever.
   - Mitigation: TTL expiration + heartbeat/refresh + reclaim command.

## Gaps vs desired product

- No `owner` concept for leases.
- No lease persistence store.
- No release-by-owner enforcement.
- No audit trail / allocation history.
- No CLI-first workflow yet (allocate/release/list/doctor).
- No documented contract for CI/agent automation JSON output.

## Security / ops concerns

- Potential accidental exposure of host process/container metadata in diagnostics.
- Stale lock files if interrupted writes are not handled.
- User confusion if scanner reports different state than lease store.

## Required design decisions (for brainstorm)

1. Lease store backend for v1:
   - file JSON + lock vs sqlite.
2. Owner identity format:
   - explicit `--owner` string vs auto hostname/pid default.
3. TTL defaults and expiry strategy:
   - hard expiry only vs keepalive refresh command.
4. Conflict policy:
   - fail on occupied requested port vs auto-next fallback.
5. Scanner precedence:
   - lease store first, then runtime scan, then bind-probe confirmation.

## Recommended v1 acceptance criteria

- Deterministic allocation under single process.
- No duplicates under two concurrent allocators (tested).
- Lease lifecycle complete: allocate/release/list/reclaim-expired.
- CLI JSON output stable enough for Cursor/agents/CI consumption.
- Works without Docker installed.
