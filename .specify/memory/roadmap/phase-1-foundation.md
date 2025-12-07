# Phase 1: Foundation

**Goal**: Bootstrap the swift-quiche project with proper package structure, dependency management, and QUICHE source integration.

**Status**: 🔲 Not Started  
**Target Release**: v0.0.1  
**Last Updated**: 2025-12-07

---

## Features

### 1.1 Package Bootstrap

**Purpose & User Value**: Establish the Swift package structure so developers can add swift-quiche as a dependency and the project builds on all target platforms.

**Success Metrics**:
- Package.swift parses without errors
- `swift build` succeeds on macOS (arm64, x86_64)
- `swift build` succeeds on Linux (Ubuntu 22.04)
- Package can be added as a dependency in another project

**Dependencies**: None (first feature)

**Notes**:
- Use Swift 6.1+ toolchain
- Configure for all Apple platforms + Linux
- Set up product targets: `Quiche` (high-level), `libquiche` (bindings)

---

### 1.2 swift-abseil Package

**Purpose & User Value**: Create a standalone Swift package wrapping Google's Abseil C++ library, required by QUICHE for core utilities.

**Success Metrics**:
- Separate `swift-abseil` repository created
- Builds on macOS and Linux
- CI pipeline passing
- Can be imported as SPM dependency

**Dependencies**: None (parallel with 1.1)

**Notes**:
- Extract minimal Abseil components needed by QUICHE
- Use subtree.yaml for Abseil source management
- Document which Abseil modules are included

---

### 1.3 swift-boringssl Package

**Purpose & User Value**: Create a standalone Swift package wrapping BoringSSL, required by QUICHE for TLS operations.

**Success Metrics**:
- Separate `swift-boringssl` repository created
- Builds on macOS and Linux
- CI pipeline passing
- TLS operations functional (basic test)

**Dependencies**: None (parallel with 1.1, 1.2)

**Notes**:
- Reference Apple's swift-crypto CCryptoBoringSSL for patterns
- May be able to fork/adapt existing work
- Ensure hardened TLS configuration per constitution

---

### 1.4 QUICHE Subtree Integration

**Purpose & User Value**: Integrate Google QUICHE source code into the repository using swift-plugin-subtree for reproducible builds and easy updates.

**Success Metrics**:
- `subtree.yaml` configured for QUICHE
- `subtree add` successfully pulls QUICHE source
- Vendor directory contains QUICHE at pinned commit
- Update process documented

**Dependencies**: 1.1 (Package Bootstrap)

**Notes**:
- Pin to specific QUICHE commit (not tracking main)
- Configure extraction patterns for needed source files
- Exclude test files, benchmarks, unnecessary modules

---

### 1.5 Source Extraction to libquiche Target

**Purpose & User Value**: Extract QUICHE C/C++ source files into a Swift-compatible target that can be built by SPM.

**Success Metrics**:
- `Sources/libquiche/` contains extracted QUICHE sources
- SPM can compile the C/C++ sources
- No missing headers or undefined symbols
- Builds on macOS and Linux

**Dependencies**: 1.2 (swift-abseil), 1.3 (swift-boringssl), 1.4 (Subtree Integration)

**Notes**:
- May require modulemap configuration
- Handle C++ interop carefully (Swift 6.1 C++ interop)
- Document any manual patches needed

---

### 1.6 CI Pipeline Setup

**Purpose & User Value**: Automated CI ensures code quality and cross-platform compatibility on every PR.

**Success Metrics**:
- GitHub Actions workflow configured
- Builds on macOS arm64, macOS x86_64, iOS Simulator
- Linux builds (Ubuntu 22.04 x86_64) in P2
- SwiftLint and SwiftFormat checks passing

**Dependencies**: 1.1 (Package Bootstrap)

**Notes**:
- Start with P1 platforms only
- Add Linux in Phase 2 (may have build complexity)
- Configure branch protection rules

---

## Phase Dependencies & Sequencing

```
1.1 Package Bootstrap ──┬──► 1.4 Subtree Integration ──► 1.5 Source Extraction
                        │
                        └──► 1.6 CI Pipeline Setup
                        
1.2 swift-abseil ───────┬──► 1.5 Source Extraction
                        │
1.3 swift-boringssl ────┘
```

**Critical Path**: 1.1 → 1.4 → 1.5 (with 1.2, 1.3 as parallel dependencies for 1.5)

---

## Phase Metrics

| Metric | Target |
|--------|--------|
| Build success (macOS) | 100% |
| Build success (iOS Simulator) | 100% |
| CI pipeline green | All checks pass |
| Dependencies integrated | swift-abseil, swift-boringssl functional |

---

## Phase Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Abseil build complexity | High | Start minimal; add modules as needed |
| BoringSSL SPM integration | Medium | Reference swift-crypto patterns |
| QUICHE extraction complexity | High | Iterate on extraction patterns; accept manual patches initially |

---

## Next Specs

After completing this phase, proceed to Phase 2 with:
- `/speckit.specify "Feature: QUIC Connection API — basic connection lifecycle with async/await"`
- `/speckit.specify "Feature: Echo Sample — simple QUIC echo client and server"`
