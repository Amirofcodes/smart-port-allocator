# Smart Port Allocator

Reliable local port allocation for developers, CLIs, and automation.

## Why this exists

Local dev gets noisy fast: multiple services, duplicate defaults, and repeated `address already in use` failures.

Smart Port Allocator provides a predictable way to:
- discover free ports,
- reserve them safely,
- track leases,
- and prevent collisions across parallel projects.

## Scope (v1)

- Free-port discovery with configurable ranges
- Lease/reservation model (TTL + release)
- Conflict-safe allocation for concurrent processes
- CLI-first UX for local workflows and scripts

## Status

Early build / productization stage.

This project is the focused evolution of the Smart Port Allocation capabilities initially developed in [ChimeraStack_CLI](https://github.com/Amirofcodes/ChimeraStack_CLI).

## Near-term roadmap

1. Core allocator package (Python)
2. Stable CLI (`allocate`, `release`, `list`, `doctor`)
3. Locking strategy for concurrency safety
4. JSON output for CI/agent automation
5. Integration examples (Docker Compose, local scripts)

## Contributing

Contributions and design feedback are welcome once v1 API is stabilized.

## License

MIT
