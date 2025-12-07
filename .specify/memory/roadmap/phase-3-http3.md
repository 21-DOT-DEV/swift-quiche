# Phase 3: HTTP/3

**Goal**: Implement the HTTP/3 layer on top of Core QUIC, providing high-level client and server APIs for HTTP requests/responses.

**Status**: 🔲 Not Started  
**Target Release**: v0.2.0  
**Last Updated**: 2025-12-07

---

## Features

### 3.1 HTTP3 Request/Response Types

**Purpose & User Value**: Provide Swift-native types for HTTP/3 requests and responses that feel familiar to developers using URLSession or similar APIs.

**Success Metrics**:
- `HTTP3.Request` type with method, URL, headers, body
- `HTTP3.Response` type with status, headers, body
- Header handling follows HTTP semantics (case-insensitive keys)
- Body supports streaming (AsyncSequence)

**Dependencies**: Phase 2 complete

**Notes**:
- Consider alignment with swift-http-types if beneficial
- Support common HTTP methods (GET, POST, PUT, DELETE, etc.)
- Handle HTTP/3-specific headers (pseudo-headers)

---

### 3.2 HTTP3 Client API

**Purpose & User Value**: Enable developers to make HTTP/3 requests with a simple, high-level async API.

**Success Metrics**:
- `HTTP3.Client` type with `fetch(_:)` method
- Automatic connection management (reuse, pooling)
- Request/response streaming works
- Timeout and cancellation supported

**Dependencies**: 3.1 (Request/Response Types)

**Notes**:
- Connection pooling can be deferred (single connection first)
- Follow redirects optionally
- Expose underlying QUIC.Connection for advanced use

---

### 3.3 HTTP3 Server API

**Purpose & User Value**: Enable developers to handle incoming HTTP/3 requests and send responses on the server side.

**Success Metrics**:
- `HTTP3.Server` type with request handler callback/closure
- Async handler pattern: `(HTTP3.Request) async throws -> HTTP3.Response`
- Multiple concurrent requests handled
- Graceful shutdown supported

**Dependencies**: 3.1 (Request/Response Types), Phase 2 Server API

**Notes**:
- Router/middleware is out of scope (users add their own)
- Focus on correctness, not performance optimization yet
- Support HTTP/3 server push as optional feature

---

### 3.4 QPACK Integration

**Purpose & User Value**: Ensure HTTP/3 header compression (QPACK) works correctly, handled transparently by the API.

**Success Metrics**:
- QPACK encoding/decoding handled by QUICHE
- No head-of-line blocking issues
- Header table size configurable
- Large header sets work correctly

**Dependencies**: 3.1 (Request/Response Types)

**Notes**:
- QPACK is internal — users don't interact directly
- Validate against RFC 9204 requirements
- Monitor for edge cases in header handling

---

### 3.5 HTTP3 Client Sample

**Purpose & User Value**: Demonstrate HTTP/3 client usage with a CLI tool that fetches URLs.

**Success Metrics**:
- `http3-client` CLI fetches URLs and prints response
- Works with public HTTP/3 endpoints (google.com, cloudflare.com)
- Displays headers, status, and body
- Error handling shows useful messages

**Dependencies**: 3.2 (Client API)

**Notes**:
- Similar to `curl` for HTTP/3
- Include timing/performance output option
- Document usage in README

---

### 3.6 HTTP3 Server Sample

**Purpose & User Value**: Demonstrate HTTP/3 server usage with a simple server that handles requests.

**Success Metrics**:
- `http3-server` CLI serves static responses
- Handles multiple concurrent clients
- TLS certificate configuration documented
- Works on macOS and Linux

**Dependencies**: 3.3 (Server API)

**Notes**:
- Simple "Hello, World" or echo server
- Include example with file serving
- Document TLS setup requirements

---

### 3.7 QuicheDemo iOS/macOS App

**Purpose & User Value**: Provide a SwiftUI demo app showing HTTP/3 in action, demonstrating mobile/desktop integration.

**Success Metrics**:
- SwiftUI app builds for iOS and macOS
- Fetches URL and displays response
- Shows connection state and metrics
- Demonstrates proper async/await usage

**Dependencies**: 3.2 (Client API)

**Notes**:
- Visual demo of connection lifecycle
- Show RTT, bytes transferred, etc.
- Useful for presentations and documentation

---

### 3.8 DocC Documentation

**Purpose & User Value**: Comprehensive API documentation for all public types and methods, enabling developers to use swift-quiche effectively.

**Success Metrics**:
- 100% public API coverage
- Code examples in documentation
- Conceptual articles for getting started
- Hosted on Swift Package Index or GitHub Pages

**Dependencies**: 3.2 (Client API), 3.3 (Server API)

**Notes**:
- Follow Apple's DocC best practices
- Include tutorials for common use cases
- Link to QUICHE upstream docs where appropriate

---

## Phase Dependencies & Sequencing

```
3.1 Request/Response Types ──┬──► 3.2 Client API ──┬──► 3.5 Client Sample
                             │                     │
                             │                     └──► 3.7 QuicheDemo App
                             │
                             ├──► 3.3 Server API ──────► 3.6 Server Sample
                             │
                             └──► 3.4 QPACK Integration

All features ──► 3.8 DocC Documentation
```

**Critical Path**: 3.1 → 3.2 → 3.5 (client path) and 3.1 → 3.3 → 3.6 (server path)

---

## Phase Metrics

| Metric | Target |
|--------|--------|
| Public endpoints reachable | google.com, cloudflare.com |
| Server handles concurrent requests | ≥10 simultaneous |
| DocC coverage | 100% public API |
| Sample apps functional | iOS, macOS, Linux CLI |

---

## Phase Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| HTTP/3 semantics complexity | Medium | Follow RFC 9114 strictly; test with real servers |
| QPACK edge cases | Medium | Use QUICHE's implementation; add targeted tests |
| iOS app signing/provisioning | Low | Document requirements; use simulator for CI |

---

## Next Specs

After completing this phase, proceed to Phase 4 with:
- `/speckit.specify "Feature: OHTTP Client — Oblivious HTTP request encapsulation"`
- `/speckit.specify "Feature: CONNECT-UDP — UDP proxying over HTTP/3"`
