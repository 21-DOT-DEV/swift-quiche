# Phase 2: Core QUIC

**Goal**: Implement the foundational QUIC Swift API with connection lifecycle, stream management, and basic client/server functionality.

**Status**: 🔲 Not Started  
**Target Release**: v0.1.0  
**Last Updated**: 2025-12-07

---

## Features

### 2.1 QUIC Configuration API

**Purpose & User Value**: Provide a Swift-native way to configure QUIC connections with safe defaults and explicit opt-in for advanced options.

**Success Metrics**:
- `QUIC.Configuration` type with builder pattern or struct
- Safe defaults per constitution (timeouts, flow control, max datagram size)
- Validation rejects invalid configurations
- DocC documentation complete

**Dependencies**: Phase 1 complete

**Notes**:
- Follow Swift API Design Guidelines
- Unsafe options require explicit naming (e.g., `unsafeDisableValidation`)
- Configuration immutable after connection creation

---

### 2.2 QUIC Connection API

**Purpose & User Value**: Enable developers to establish QUIC connections using async/await, with proper lifecycle management (connect, streams, close).

**Success Metrics**:
- `QUIC.Connection` type with async `connect()`, `close()`
- Connection state observable (connecting, connected, closing, closed)
- Proper cleanup on deinit (no resource leaks)
- Unit tests for lifecycle states

**Dependencies**: 2.1 (Configuration API)

**Notes**:
- Client-initiated connections first
- Server accept comes in 2.4
- Handle connection migration events (expose, don't implement custom logic)

---

### 2.3 QUIC Stream API

**Purpose & User Value**: Allow developers to open, send, receive, and close QUIC streams with backpressure handling.

**Success Metrics**:
- `QUIC.Stream` type with async `send(_:)`, `receive()`, `close()`
- Bidirectional and unidirectional stream support
- Flow control respected (backpressure works)
- Stream cancellation supported

**Dependencies**: 2.2 (Connection API)

**Notes**:
- AsyncSequence for receiving data
- Handle stream reset and stop-sending
- Expose stream ID for debugging

---

### 2.4 QUIC Server/Listener API

**Purpose & User Value**: Enable developers to accept incoming QUIC connections on the server side.

**Success Metrics**:
- `QUIC.Listener` type with async `accept()` returning connections
- Configurable bind address and port
- Proper TLS certificate configuration
- Unit tests for accept lifecycle

**Dependencies**: 2.2 (Connection API)

**Notes**:
- AsyncSequence pattern for accepting connections
- Handle listener shutdown gracefully
- Support multiple concurrent connections

---

### 2.5 Error Handling

**Purpose & User Value**: Provide clear, actionable Swift errors for QUIC operations without leaking internal details.

**Success Metrics**:
- `QUICError` enum with cases for common failures
- Error descriptions are user-friendly and actionable
- QUIC transport error codes mapped to Swift errors
- No sensitive data in error descriptions

**Dependencies**: 2.2 (Connection API)

**Notes**:
- Map QUICHE error codes to Swift errors
- Include connection/stream context where helpful
- Follow constitution: errors don't leak secrets

---

### 2.6 Echo Sample Project

**Purpose & User Value**: Demonstrate basic QUIC functionality with a simple echo client and server that developers can run and modify.

**Success Metrics**:
- `quiche-echo-client` CLI sends message, receives echo
- `quiche-echo-server` CLI echoes received messages
- Works on macOS (arm64, x86_64)
- README with usage instructions

**Dependencies**: 2.3 (Stream API), 2.4 (Server API)

**Notes**:
- Minimal dependencies (stdlib + swift-quiche)
- Self-signed certificates for demo
- Foundation for later samples

---

### 2.7 Linux CI Integration

**Purpose & User Value**: Ensure swift-quiche builds and tests pass on Linux for server deployment use cases.

**Success Metrics**:
- CI builds on Ubuntu 22.04 (x86_64)
- CI builds on Ubuntu 22.04 (arm64) if runner available
- All unit tests pass on Linux
- Echo sample works on Linux

**Dependencies**: 2.6 (Echo Sample)

**Notes**:
- May require platform-specific code paths
- Document any Linux-specific setup requirements

---

## Phase Dependencies & Sequencing

```
2.1 Configuration API ──► 2.2 Connection API ──┬──► 2.3 Stream API ──► 2.6 Echo Sample ──► 2.7 Linux CI
                                               │
                                               └──► 2.4 Server API ──────┘
                                               │
                                               └──► 2.5 Error Handling
```

**Critical Path**: 2.1 → 2.2 → 2.3 → 2.6

---

## Phase Metrics

| Metric | Target |
|--------|--------|
| Unit test coverage | ≥60% |
| Integration tests | Connection lifecycle, stream ops |
| Memory leaks | Zero in FFI boundary |
| Echo sample functional | macOS + Linux |

---

## Phase Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Async/await + FFI complexity | High | Start with blocking wrapper, add async layer |
| Memory management at boundary | High | Extensive testing, leak detection in CI |
| Linux platform differences | Medium | Test early and often |

---

## Next Specs

After completing this phase, proceed to Phase 3 with:
- `/speckit.specify "Feature: HTTP3 Client — fetch URLs over HTTP/3 with async/await"`
- `/speckit.specify "Feature: HTTP3 Server — handle HTTP/3 requests"`
