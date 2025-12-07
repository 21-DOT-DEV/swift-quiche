<!--
Sync Impact Report:
- Version: N/A → 1.0.0 (Initial constitution)
- Change Type: Initial creation
- Scope: swift-quiche package (/Users/csjones/Developer/swift-quiche)
- Structure: Two-tier (7 core principles + implementation guidance)
- Core Principles:
  I. Protocol Compliance & Interoperability
  II. Security by Design
  III. Performance & Resource Safety
  IV. API Design & Safety
  V. Spec-First & Test-Driven Development
  VI. Cross-Platform CI & Quality Gates
  VII. Open Source Excellence
- Enforcement: Three-tier model (MUST/SHOULD/MAY)
- Governance: Minimal model (project owner amends directly)
- Compliance: Continuous CI + event-driven review
- Templates Status:
  ⚠ spec-template.md - Requires alignment review
  ⚠ plan-template.md - Requires alignment review
  ⚠ tasks-template.md - Requires alignment review
  ⚠ checklist-template.md - Requires alignment review
- Follow-up TODOs:
  • Create SECURITY.md with upstream deferral + local wrapper handling
  • Create CONTRIBUTING.md with development workflow
-->

# Constitution for swift-quiche

## Preamble

This constitution governs the **swift-quiche** package, a Swift wrapper around Google's QUICHE library providing QUIC, HTTP/3, and related protocol support.

**Scope**: This repository only. Covers both the low-level C++ bindings (`libquiche`) and the high-level Swift API (`Quiche`).

**Philosophy**: Principles are technology-agnostic where possible. This package wraps battle-tested upstream code—correctness, safety, and interoperability take precedence over features.

---

## Core Principles

### I. Protocol Compliance & Interoperability

**Statement**: QUIC, HTTP/3, and QPACK behavior MUST strictly follow IETF RFCs. The Swift wrapper MUST correctly expose QUICHE's protocol-compliant behavior without divergence.

**Rationale**: QUIC is extremely sensitive to non-standard behavior; interoperability requires exact compliance. The package wraps QUICHE, which handles protocol correctness—our job is to expose it faithfully.

**Practices**:
- **MUST** preserve QUICHE's RFC-compliant behavior for QUIC (RFC 9000), HTTP/3 (RFC 9114), and QPACK (RFC 9204)
- **MUST** maintain strict state machine discipline: connection lifecycle, packet number spaces, ACK logic, loss recovery
- **MUST NOT** modify QUIC transport invariants (long header structure, version negotiation)
- **MUST NOT** implement custom protocol extensions outside QUICHE's supported features
- **SHOULD** validate Swift wrapper behavior against upstream QUICHE test expectations
- **SHOULD** run smoke interop tests against public QUIC endpoints (scheduled, not PR-blocking)
- **MAY** expose QUICHE's experimental features via Package Traits (off by default)

**Compliance**: Integration tests verify wrapper correctly calls QUICHE APIs. Scheduled smoke tests validate external connectivity.

---

### II. Security by Design

**Statement**: The package MUST use QUICHE with hardened TLS (BoringSSL) and preserve all recommended QUIC security practices. The Swift wrapper MUST NOT introduce security vulnerabilities.

**Rationale**: QUIC shifts significant security responsibility to the transport layer. QUICHE handles cryptographic correctness—our wrapper must not weaken it through misuse, memory safety issues, or resource exhaustion.

**Practices**:
- **MUST** preserve QUICHE's security features: anti-amplification, retry token validation, handshake confirmation, key rotation
- **MUST** use safe Swift patterns for C++ interop (no dangling pointers, proper lifetime management)
- **MUST** validate all inputs crossing the Swift/C++ boundary
- **MUST** prevent resource exhaustion (unbounded buffer growth, connection floods)
- **MUST NOT** expose raw cryptographic material through public Swift APIs
- **MUST NOT** weaken default TLS configuration
- **SHOULD** use Swift's memory safety features (non-optional where appropriate, bounds checking)
- **SHOULD** document security-relevant configuration options clearly
- **MAY** provide unsafe escape hatches for expert users, clearly documented as such

**Compliance**: Code review verifies memory safety at FFI boundaries. CI scans for obvious vulnerabilities.

---

### III. Performance & Resource Safety

**Statement**: The package MUST yield stable latency and throughput by correctly exposing QUICHE's congestion control, pacing, and flow control. Resource usage MUST be bounded and predictable.

**Rationale**: Performance is a core reason QUIC exists. QUICHE provides battle-tested algorithms—our wrapper must not introduce bottlenecks, leaks, or unbounded resource consumption.

**Practices**:
- **MUST** correctly expose QUICHE's congestion control (BBR, CUBIC, Reno) without modification
- **MUST** enforce stream-level and connection-level flow control as QUICHE specifies
- **MUST** prevent memory leaks across the FFI boundary
- **MUST** bound buffer sizes and connection counts with configurable limits
- **MUST** handle backpressure correctly (don't queue unbounded data)
- **SHOULD** prefer zero-copy APIs for I/O paths where QUICHE supports them
- **SHOULD** minimize allocations in hot paths
- **SHOULD** expose QUIC metrics (RTT, congestion window, loss rate, bytes sent/received)
- **MAY** provide platform-specific optimizations where beneficial

**Compliance**: Integration tests verify resource cleanup. Performance benchmarks track regressions.

---

### IV. API Design & Safety

**Statement**: Public APIs MUST follow Swift-native conventions with safe defaults. The package provides two tiers: low-level bindings for advanced users and a high-level async/await API for typical use cases.

**Rationale**: Familiar API design lowers learning curve and reduces misuse. Safe defaults protect typical users while supporting advanced protocols for experts. Async/await is idiomatic Swift 6.

**Practices**:
- **MUST** use protocol-aligned namespaces: `QUIC.Connection`, `HTTP3.Stream`
- **MUST** follow Swift API Design Guidelines (clear naming, grammatical methods)
- **MUST** provide async/await as the primary high-level API
- **MUST** use strongly-typed errors with actionable messages
- **MUST** ensure default configurations are safe (sensible timeouts, flow control limits, max datagram sizes)
- **MUST** require explicit opt-in for unsafe or experimental features
- **MUST NOT** expose implementation details that could be misused
- **SHOULD** separate high-level Swift API from low-level bindings as distinct products
- **SHOULD** document all public types and functions with DocC
- **MAY** provide callback-based APIs for users needing lower-level control

**Compliance**: Code review enforces API naming conventions. DocC coverage required for public APIs.

---

### V. Spec-First & Test-Driven Development

**Statement**: Every feature MUST start with a specification. All code MUST follow test-driven development: tests written first, verified to fail, then implementation proceeds.

**Rationale**: Specifications ensure alignment with user needs and provide measurable success criteria. TDD prevents regressions, enables confident refactoring, and documents expected behavior.

**Practices - Specification Requirements**:
- **MUST** create `spec.md` for every feature before development
- **MUST** represent a single feature or small subfeature (not multiple unrelated features)
- **MUST** be independently testable (no dependencies on incomplete specs)
- **MUST** define user scenarios, acceptance criteria, and success metrics
- **MUST NOT** combine multiple unrelated features in one spec
- **MUST NOT** describe implementation details instead of user-facing behavior

**Practices - Test-Driven Development**:
- **MUST** write tests before implementation (red → green → refactor)
- **MUST** verify tests fail initially
- **MUST** maintain separate unit and integration tests
- **SHOULD** develop outside-in (user's perspective first)
- **SHOULD** include fuzzing for FFI boundary code (scheduled CI)

**Compliance**: PRs MUST include tests written first. CI blocks merges if tests missing or immediately passing.

---

### VI. Cross-Platform CI & Quality Gates

**Statement**: The package MUST maintain support for all advertised platforms with CI coverage ensuring features compile and tests pass on each.

**Rationale**: Cross-platform reliability is a core value proposition. Swift 6.1+ enables broader platform support; CI ensures consistency.

**Practices**:
- **MUST** test on all supported Apple platforms (iOS, macOS, tvOS, watchOS, visionOS)
- **MUST** test on Linux (Ubuntu LTS)
- **MUST** test on both arm64 and x86_64 architectures where applicable
- **MUST** require Swift 6.1+ as minimum version
- **MUST** pass all unit and integration tests before merge
- **MUST** pass linting checks (SwiftLint, SwiftFormat) before merge
- **MUST NOT** merge code that breaks any supported platform
- **SHOULD** track Android, WASM, and Windows as aspirational targets
- **SHOULD** run fuzzing on schedule for FFI boundary code
- **MAY** provide platform-specific optimizations

**Compliance**: CI pipeline enforces all MUST-level gates. Platform failures block merge.

---

### VII. Open Source Excellence

**Statement**: All development MUST follow open source best practices: comprehensive documentation, welcoming contributions, clear licensing, and simplicity over cleverness.

**Rationale**: Good documentation reduces friction. Clear decisions preserve knowledge. Simplicity encourages contributions and reduces maintenance burden.

**Practices**:
- **MUST** maintain clear README with setup instructions and usage examples
- **MUST** provide DocC documentation for high-level Swift API
- **MUST** provide contribution guidelines (CONTRIBUTING.md)
- **MUST** include LICENSE file
- **MUST** write clear, human-readable code (readability over cleverness)
- **MUST** apply KISS and DRY principles
- **MUST** maintain zero runtime dependencies beyond QUICHE/BoringSSL
- **SHOULD** document architecture decisions
- **SHOULD** maintain security disclosure process (SECURITY.md)
- **SHOULD** provide issue and PR templates
- **MAY** offer optional SwiftNIO integration as separate product

**Compliance**: PRs MUST include documentation updates for new features or API changes.

---

## Implementation Guidance

### Two-Tier Library Architecture

**Structure**:

| Product | Type | Description |
|---------|------|-------------|
| `Quiche` | Swift API | High-level async/await API (primary) |
| `libquiche` | C++ bindings | Low-level bindings for advanced users |

**Workflow**:
- High-level API (`Quiche`) is the recommended entry point
- Low-level bindings (`libquiche`) for users needing direct QUICHE access
- Both products versioned together in the same package

---

### Package Traits for Extensions

**Statement**: QUIC extensions (datagrams, WebTransport, MASQUE) MUST be gated behind Package Traits, disabled by default.

**Configuration**:
```swift
traits: [
    .default(enabledTraits: ["QUIC", "HTTP3"]),
    .trait(name: "Datagrams", description: "QUIC Datagram extension (RFC 9221)"),
    .trait(name: "WebTransport", description: "WebTransport over HTTP/3"),
    .trait(name: "MASQUE", description: "MASQUE proxy protocol"),
]
```

**Requirements**:
- **MUST** keep extensions off by default
- **MUST** use `#if` compilation for trait-gated code
- **MUST** document each trait's purpose and stability
- **SHOULD** follow upstream QUICHE's support status for extensions

---

### Security Disclosure Process

**Statement**: Security issues MUST be handled according to their scope.

**Scope Division**:
- **Protocol/cryptographic issues** → Direct to Google's QUICHE security process
- **Swift wrapper issues** (memory safety, resource exhaustion, unsafe API misuse) → Handle locally

**Local Process Requirements**:
- **MUST** provide SECURITY.md with reporting instructions
- **MUST** include contact method for wrapper-specific issues
- **MUST** acknowledge reports within 48 hours
- **MUST** follow coordinated disclosure (90-day standard)
- **SHOULD** document scope clearly (wrapper vs upstream)

---

### Observability

**Statement**: The package SHOULD expose QUIC metrics for debugging and monitoring.

**Recommended Metrics**:
- Connection state and lifetime
- RTT estimates (smoothed, variance)
- Congestion window size
- Bytes sent/received
- Packet loss rate
- Stream counts (open, closed)
- PTO events

**Requirements**:
- **SHOULD** expose metrics via structured API
- **SHOULD** support structured logging for connection events
- **MAY** integrate with swift-metrics for server use cases

---

## Technology Stack (Current Implementation)

**Note**: Constitution defines technology-agnostic principles. This section documents current choices, which may change without constitutional amendments.

### Supported Platforms

**Required**:
- iOS (arm64)
- macOS (arm64, x86_64)
- tvOS (arm64)
- watchOS (arm64)
- visionOS (arm64)
- Linux (x86_64, arm64)

**Aspirational**:
- Android
- WebAssembly (WASM)
- Windows

### Current Stack

**Language**: Swift 6.1+
**Build**: Swift Package Manager (SPM)
**Upstream**: QUICHE (Google), BoringSSL
**Testing**: swift-testing, XCTest
**Documentation**: DocC
**Linting**: SwiftLint, SwiftFormat
**CI**: GitHub Actions

### Dependencies

**Runtime**: Zero dependencies beyond QUICHE/BoringSSL bindings
**Optional**: SwiftNIO integration (separate product, not core)
**Development only**: SwiftLint, SwiftFormat

---

## Governance

### Authority

This constitution supersedes all other development practices. Deviations MUST be explicitly justified and approved.

**Model**: Project owner can amend constitution directly with rationale.

### Amendment Process

1. Project owner proposes amendment with rationale and impact analysis
2. Version updated (semantic versioning):
   - **MAJOR**: Backward-incompatible changes or principle removals
   - **MINOR**: New principle or materially expanded guidance
   - **PATCH**: Clarifications, wording fixes
3. Update dependent templates in `.specify/templates/`
4. Document changes in Sync Impact Report
5. Commit with descriptive message

### Compliance Review Triggers

| Trigger | Action |
|---------|--------|
| QUICHE upstream version bump | Verify behavior consistency |
| New platform support | Full CI coverage validation |
| FFI boundary changes | Memory safety review required |
| Breaking API changes (semver major) | Stability signaling review |

### Versioning & Stability

**Pre-1.0** (current):
- No stability guarantees
- Breaking changes acceptable with minor version bumps
- Users advised to pin exact versions

**Post-1.0** (future):
- Semantic versioning strictly enforced
- Deprecation period before removal
- Breaking changes require major version bump

### Enforcement

- PR reviewers verify constitutional alignment
- CI pipeline enforces MUST-level (blocking), SHOULD-level (warnings)
- Three-tier enforcement:
  - **MUST**: Blocks merge
  - **SHOULD**: Warning, requires override justification
  - **MAY**: Informational only

---

## Version History

**Version**: 1.0.0
**Ratified**: 2025-12-07
**Last Amended**: 2025-12-07

**Changelog**:
- **1.0.0** (2025-12-07): Initial constitution with 7 core principles, three-tier enforcement, minimal governance, two-tier library architecture, Package Traits for extensions.

---

## Appendix: Principle Mapping

This constitution consolidates principles from the provided sources:

**From QUIC/HTTP3 Principle Table**:
- Protocol Compliance → Principle I
- Interop-First Development → Principle I
- Security-by-Design → Principle II
- State Machine Correctness → Principles I, II
- Transport Invariants → Principle I
- Predictable Performance → Principle III
- Congestion Control Safety → Principle III
- Loss Recovery Correctness → Principle I (upstream)
- Connection Migration Robustness → Principle I (upstream)
- QPACK Safety & Determinism → Principle I (upstream)
- HTTP/3 Layer Discipline → Principle I
- Flow Control Correctness → Principle III
- Robust Error Handling → Principle IV
- Resource Safety → Principle III
- Observability → Implementation Guidance
- Test Strategy → Principle V
- Upgrade & Versioning Discipline → Governance
- Configuration Safety → Principle IV
- Zero-Copy & Performance APIs → Principle III
- Asynchronous Runtime Separation → Principle IV
- Extension Cleanliness → Implementation Guidance (Package Traits)

**From Core Principles (4.1)**:
- Spec-First & Outside-In → Principle V
- Test-Driven Development → Principle V
- Small, Independent Specs → Principle V
- CI & Quality Gates → Principle VI
- Simplicity & Readability → Principle VII
- Open Source Excellence → Principle VII
- Governance & Amendments → Governance

**From Security Principles**:
- Secure Random Number Generation → Principle II (upstream)
- Key & Data Protection → Principle II
- Error Handling → Principle IV
- Cross-Platform CI → Principle VI
