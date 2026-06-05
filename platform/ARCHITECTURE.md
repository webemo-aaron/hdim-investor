# Architecture

## Summary

HDIM should be described technically as a standards-native, multi-service healthcare platform with event-driven patterns, gateway-mediated ingress, and shared platform controls. The strongest architecture claims are the ones that can be traced back to generated inventories and code-validated architecture documents.

## Core Architecture Themes

### 1. Standards-native healthcare execution

HDIM uses FHIR-centered service patterns and interoperability services rather than reducing the public technical story to a proprietary internal data model.

### 2. Event-driven operating model

Key domains use event-driven and event-sourced patterns. This supports replay, projection, auditability, and more explicit change flows than a purely CRUD-oriented service estate.

### 3. Shared control layers

Shared platform modules cover:

- authentication
- authorization and security
- audit
- tracing
- persistence
- messaging
- API-contract support

These shared controls matter because they show platform-level standardization across a broad service estate.

## Platform Topology

```text
External Edge
  -> Domain Gateways
    -> Clinical / Patient / Quality / Interoperability / Analytics / Workflow Services
      -> Shared Security / Audit / Tracing / Persistence / Messaging
```

This public summary intentionally stays at the topology level. Deeper controller, contract, and runtime dependency analysis belongs in the NDA diligence package.

## Why The Architecture Is Credible

The technical story is stronger because it is anchored to code-validated inventory and review work:

- generated service inventory exists
- generated dependency mapping exists
- shared module inventory exists
- contract audit and remediation queue exist internally

## Explicit Technical Limits

The current public story should also be transparent about limits:

- not every internal API is published as a public contract artifact
- contract governance is improving but not yet uniformly complete
- deeper runtime and readiness analysis should be treated as NDA-protected

## Review Path

For public review:

1. Read this file
2. Read [PLATFORM-OVERVIEW.md](PLATFORM-OVERVIEW.md)
3. Read [../traction/PRODUCTION-READINESS.md](../traction/PRODUCTION-READINESS.md)

For deeper diligence, move to the NDA package rather than overloading this public repo with internal architecture detail.
