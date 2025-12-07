# Phase 5: Extensions

**Goal**: Implement QUIC/HTTP/3 extensions as optional Package Traits: Datagrams, WebTransport, MASQUE, and other experimental features.

**Status**: 🔲 Not Started  
**Target Release**: v0.9.0  
**Last Updated**: 2025-12-07

---

## Features

### 5.1 Package Traits Infrastructure

**Purpose & User Value**: Set up Swift Package Traits (SE-0450) to allow users to opt-in to experimental features without bloating the default build.

**Success Metrics**:
- Package.swift configured with traits
- Default build excludes extensions
- `--traits` flag enables specific extensions
- Conditional compilation works (`#if Datagrams`)

**Dependencies**: Phase 4 complete

**Notes**:
- Traits: `Datagrams`, `WebTransport`, `MASQUE`
- Default traits: `QUIC`, `HTTP3` only
- Document trait usage in README

---

### 5.2 QUIC Datagrams (RFC 9221)

**Purpose & User Value**: Enable unreliable datagram transmission over QUIC for latency-sensitive applications (gaming, real-time media).

**Success Metrics**:
- `QUIC.Connection.sendDatagram(_:)` method
- `QUIC.Connection.receiveDatagram()` async method
- Max datagram size respected
- Works only when trait enabled

**Dependencies**: 5.1 (Package Traits)

**Notes**:
- Requires QUICHE datagram support
- Negotiated during handshake
- No delivery guarantees (unreliable)

---

### 5.3 WebTransport Client

**Purpose & User Value**: Enable web-style bidirectional communication over HTTP/3 for applications needing WebSocket-like semantics with QUIC performance.

**Success Metrics**:
- `WebTransport.Client` type
- Bidirectional stream support
- Datagram support (uses 5.2)
- Session establishment and teardown

**Dependencies**: 5.2 (Datagrams)

**Notes**:
- Draft standard (may change)
- Popular for gaming, real-time apps
- Browser interop is a goal

---

### 5.4 WebTransport Server

**Purpose & User Value**: Enable servers to accept WebTransport connections, supporting real-time applications.

**Success Metrics**:
- `WebTransport.Server` type
- Accepts WebTransport sessions
- Handles multiple concurrent sessions
- Stream and datagram support

**Dependencies**: 5.3 (WebTransport Client)

**Notes**:
- Complements HTTP/3 server
- May share infrastructure with Phase 3

---

### 5.5 MASQUE Complete Implementation

**Purpose & User Value**: Complete the MASQUE protocol suite beyond CONNECT-UDP/IP with additional proxy modes.

**Success Metrics**:
- Full MASQUE API coverage
- DNS-over-HTTPS integration option
- Proxy discovery mechanisms
- Performance optimizations

**Dependencies**: Phase 4 (Private Relay), 5.1 (Package Traits)

**Notes**:
- MASQUE is evolving; track drafts
- Focus on production-ready subset
- Document experimental vs stable features

---

### 5.6 HTTP/3 Priority Hints

**Purpose & User Value**: Allow clients and servers to signal request priorities, improving page load performance and resource allocation.

**Success Metrics**:
- Priority header support (RFC 9218)
- Urgency and incremental parameters
- Server respects client priorities
- Configurable priority strategies

**Dependencies**: Phase 3 (HTTP/3)

**Notes**:
- Transparent to most users
- Benefits complex applications (browsers, CDNs)
- Low priority for initial implementation

---

### 5.7 Extension Samples

**Purpose & User Value**: Demonstrate extension features with working samples.

**Success Metrics**:
- Datagram echo sample (latency demo)
- WebTransport chat sample
- Each extension has at least one sample

**Dependencies**: 5.2-5.6

**Notes**:
- Include in Examples/ directory
- Document trait requirements
- Performance benchmarks where appropriate

---

### 5.8 Pre-1.0 Stabilization

**Purpose & User Value**: Prepare the package for 1.0 release with API review, documentation completion, and stability guarantees.

**Success Metrics**:
- API review completed (breaking changes identified)
- All public APIs documented
- Migration guide from 0.x to 1.0
- Performance benchmarks established

**Dependencies**: All features complete

**Notes**:
- Deprecate experimental APIs or stabilize them
- Establish baseline performance metrics
- Community feedback incorporated

---

## Phase Dependencies & Sequencing

```
5.1 Package Traits ──┬──► 5.2 Datagrams ──► 5.3 WebTransport Client ──► 5.4 WebTransport Server
                     │
                     ├──► 5.5 MASQUE Complete
                     │
                     └──► 5.6 Priority Hints

5.2-5.6 ──► 5.7 Extension Samples

All ──► 5.8 Pre-1.0 Stabilization
```

**Critical Path**: 5.1 → 5.2 → 5.3 → 5.8

---

## Phase Metrics

| Metric | Target |
|--------|--------|
| Package Traits functional | All traits work correctly |
| Extension APIs documented | 100% coverage |
| Samples complete | One per extension |
| API stability | No breaking changes after v0.9.0 without deprecation |

---

## Phase Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Draft standard changes | Medium | Pin to specific draft versions |
| Package Traits edge cases | Low | Test early with simple traits |
| Scope creep in extensions | Medium | Strict feature prioritization |

---

## Post-Phase: 1.0 Release

After Phase 5 completion:
1. Final API review
2. Security audit (optional but recommended)
3. Performance documentation
4. Release v1.0.0
5. Begin post-1.0 roadmap (new features, optimizations)
