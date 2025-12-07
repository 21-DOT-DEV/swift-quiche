# Phase 4: Private Relay

**Goal**: Implement the core protocols for Apple iCloud Private Relay-style architecture: Oblivious HTTP (OHTTP), CONNECT-UDP, and CONNECT-IP.

**Status**: 🔲 Not Started  
**Target Release**: v0.5.0  
**Last Updated**: 2025-12-07

---

## Features

### 4.1 Oblivious HTTP (OHTTP) Client

**Purpose & User Value**: Enable clients to send HTTP requests through a relay without revealing the request content to the relay (request encapsulation).

**Success Metrics**:
- `OHTTP.Client` type with `encapsulate(_:)` method
- Request encryption using relay's public key
- Response decapsulation works correctly
- Validates against RFC 9458

**Dependencies**: Phase 3 complete

**Notes**:
- OHTTP uses HPKE (Hybrid Public Key Encryption)
- May need swift-crypto or BoringSSL HPKE support
- Key configuration from relay discovery

---

### 4.2 Oblivious HTTP (OHTTP) Server

**Purpose & User Value**: Enable servers to act as OHTTP relays or targets, handling encapsulated requests.

**Success Metrics**:
- `OHTTP.Server` type for relay and target roles
- Request decapsulation and forwarding (relay)
- Response encapsulation (target)
- Key rotation support

**Dependencies**: 4.1 (OHTTP Client)

**Notes**:
- Relay role: receives encapsulated, forwards to target
- Target role: decapsulates, processes, encapsulates response
- Support both roles in same codebase

---

### 4.3 CONNECT-UDP Client

**Purpose & User Value**: Enable clients to tunnel UDP traffic through an HTTP/3 proxy using the CONNECT-UDP method (RFC 9298).

**Success Metrics**:
- `MASQUE.ConnectUDP.Client` type
- Establishes UDP tunnel through proxy
- Sends/receives UDP datagrams through tunnel
- Handles capsule protocol correctly

**Dependencies**: Phase 3 HTTP/3 complete

**Notes**:
- Uses HTTP/3 extended CONNECT
- Requires QUIC Datagrams (may need Package Trait)
- Core of Private Relay data path

---

### 4.4 CONNECT-UDP Server

**Purpose & User Value**: Enable servers to act as CONNECT-UDP proxies, forwarding UDP traffic on behalf of clients.

**Success Metrics**:
- `MASQUE.ConnectUDP.Server` type
- Accepts CONNECT-UDP requests
- Forwards datagrams to target
- Handles multiple concurrent tunnels

**Dependencies**: 4.3 (CONNECT-UDP Client)

**Notes**:
- Ingress relay receives client connections
- Egress relay forwards to final destination
- Address validation per security requirements

---

### 4.5 CONNECT-IP Client

**Purpose & User Value**: Enable clients to tunnel IP-level traffic through an HTTP/3 proxy using the CONNECT-IP method (RFC 9484).

**Success Metrics**:
- `MASQUE.ConnectIP.Client` type
- Establishes IP tunnel through proxy
- Sends/receives IP packets through tunnel
- Route configuration supported

**Dependencies**: 4.3 (CONNECT-UDP Client)

**Notes**:
- More powerful than CONNECT-UDP (full IP)
- Used for VPN-like scenarios
- May require elevated permissions on some platforms

---

### 4.6 CONNECT-IP Server

**Purpose & User Value**: Enable servers to act as CONNECT-IP proxies, forwarding IP traffic on behalf of clients.

**Success Metrics**:
- `MASQUE.ConnectIP.Server` type
- Accepts CONNECT-IP requests
- Forwards IP packets to network
- Handles address assignment

**Dependencies**: 4.5 (CONNECT-IP Client)

**Notes**:
- More complex than UDP proxy
- Network interface management required
- Platform-specific implementation details

---

### 4.7 Private Relay Client Sample

**Purpose & User Value**: Demonstrate full Private Relay client flow — connecting through dual relays for privacy.

**Success Metrics**:
- `relay-client` CLI establishes relay connection
- Routes traffic through ingress → egress → destination
- Privacy properties maintained (no single point sees both client IP and destination)
- Performance acceptable for interactive use

**Dependencies**: 4.1 (OHTTP), 4.3 (CONNECT-UDP)

**Notes**:
- Two-hop architecture like iCloud Private Relay
- May connect to test infrastructure or public relays
- Document privacy model and limitations

---

### 4.8 Private Relay Server Sample

**Purpose & User Value**: Demonstrate full Private Relay server components — ingress and egress relays.

**Success Metrics**:
- `relay-ingress` server accepts client connections
- `relay-egress` server forwards to destinations
- Servers interoperate correctly
- Can run full relay stack locally for testing

**Dependencies**: 4.2 (OHTTP Server), 4.4 (CONNECT-UDP Server)

**Notes**:
- Separate binaries for ingress and egress
- Configuration for relay chaining
- Performance and scalability testing setup

---

## Phase Dependencies & Sequencing

```
4.1 OHTTP Client ──► 4.2 OHTTP Server ──────────────────────┐
                                                             │
4.3 CONNECT-UDP Client ──► 4.4 CONNECT-UDP Server ──────────┼──► 4.7 Relay Client Sample
                                                             │
4.5 CONNECT-IP Client ──► 4.6 CONNECT-IP Server             │
                                                             │
                                                             └──► 4.8 Relay Server Sample
```

**Critical Path**: 4.1 → 4.3 → 4.7 (minimum viable Private Relay client)

---

## Phase Metrics

| Metric | Target |
|--------|--------|
| OHTTP round-trip | Request/response works |
| CONNECT-UDP tunnel | UDP datagrams flow through |
| Relay interop | Client ↔ Ingress ↔ Egress |
| Privacy validation | No single point sees client IP + destination |

---

## Phase Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| HPKE availability | High | Verify BoringSSL HPKE support early |
| QUIC Datagrams support in QUICHE | Medium | Check QUICHE feature flags |
| Platform permissions for IP tunneling | Medium | Document requirements; may be iOS-limited |
| Complexity of multi-hop testing | Medium | Build local test infrastructure |

---

## Next Specs

After completing this phase, proceed to Phase 5 with:
- `/speckit.specify "Feature: QUIC Datagrams — unreliable datagram support via Package Trait"`
- `/speckit.specify "Feature: WebTransport — bidirectional streams over HTTP/3"`
