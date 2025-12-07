# swift-quiche Product Roadmap

**Version**: v1.0.0  
**Last Updated**: 2025-12-07  
**Status**: Active

---

## Vision & Goals

**Vision**: Provide a safe, idiomatic Swift interface to QUICHE, enabling Swift developers to build high-performance QUIC and HTTP/3 applications — from simple clients to full Private Relay infrastructure.

**Goals**:
1. **Protocol Compliance**: Expose QUICHE's RFC-compliant QUIC/HTTP/3 implementation faithfully
2. **Swift-Native Experience**: async/await APIs, strong typing, safe defaults
3. **Full Stack Support**: Both client and server use cases, including Private Relay architecture
4. **Cross-Platform**: All Apple platforms + Linux, with aspirational Android/WASM/Windows

---

## Phases Overview

| Phase | Name | Status | File | Target Release |
|-------|------|--------|------|----------------|
| 1 | Foundation | 🔲 Not Started | [phase-1-foundation.md](roadmap/phase-1-foundation.md) | v0.0.1 |
| 2 | Core QUIC | 🔲 Not Started | [phase-2-core-quic.md](roadmap/phase-2-core-quic.md) | v0.1.0 |
| 3 | HTTP/3 | 🔲 Not Started | [phase-3-http3.md](roadmap/phase-3-http3.md) | v0.2.0 |
| 4 | Private Relay | 🔲 Not Started | [phase-4-private-relay.md](roadmap/phase-4-private-relay.md) | v0.5.0 |
| 5 | Extensions | 🔲 Not Started | [phase-5-extensions.md](roadmap/phase-5-extensions.md) | v0.9.0 |

**Legend**: 🔲 Not Started | 🔄 In Progress | ✅ Complete

---

## Product-Level Metrics & Success Criteria

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Build Success** | 100% on P1 platforms | CI pass rate |
| **Test Coverage** | ≥60% for Swift code | Coverage reports |
| **API Documentation** | 100% public API coverage | DocC completeness |
| **Interop Validation** | Connect to public QUIC endpoints | Smoke test pass rate |
| **Memory Safety** | Zero leaks in FFI boundary | Leak detection in tests |
| **Sample Completeness** | Working example per major feature | Manual verification |

---

## High-Level Dependencies

```
Phase 1 (Foundation)
    │
    ├── swift-abseil (new repo)
    ├── swift-boringssl (new repo)
    └── subtree.yaml configuration
          │
          v
Phase 2 (Core QUIC)
    │
    ├── libquiche bindings
    └── QUIC.Connection API
          │
          v
Phase 3 (HTTP/3)
    │
    ├── HTTP3.Client
    └── HTTP3.Server
          │
          v
Phase 4 (Private Relay)
    │
    ├── OHTTP
    ├── CONNECT-UDP
    └── CONNECT-IP
          │
          v
Phase 5 (Extensions)
    │
    ├── Datagrams (RFC 9221)
    ├── WebTransport
    └── MASQUE
```

---

## Global Risks & Assumptions

| Risk | Impact | Mitigation |
|------|--------|------------|
| QUICHE C++ complexity | High | Start with minimal extraction; expand incrementally |
| Abseil/BoringSSL build complexity | High | Create separate packages first; validate before integrating |
| Cross-platform FFI differences | Medium | Test Linux early; use platform abstractions |
| QUICHE API instability | Medium | Pin to specific commit; document upgrade process |

**Assumptions**:
- QUICHE's C API is stable enough for production wrapping
- Swift 6.1 Package Traits work as documented
- BoringSSL can be built with SPM on all target platforms

---

## Change Log

| Version | Date | Change Type | Description |
|---------|------|-------------|-------------|
| v1.0.0 | 2025-12-07 | Initial | Initial roadmap with 5 phases targeting Private Relay architecture |
