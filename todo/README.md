# Unlayer Identity TODO

This directory is the executable roadmap for Unlayer Identity.

## Rules

- A checkbox means implemented, tested, integrated, and verified.
- Every security-sensitive feature requires tests before completion.
- Architecture changes must update IDENTITY.md and the relevant phase.
- Do not mark a phase complete while required tests are knowingly failing.
- Protocol adapters must preserve protocol semantics; do not force SIP/RADIUS/OIDC into a web-session abstraction.
- Better Auth is the authentication engine; Unlayer Identity owns platform identity, hierarchy, scope and protocol integration.
- HA must consume the generic subsystem from Unlayer Database rather than copying database-specific HA logic.

## Phases

- [ ] Phase 00 — Project foundation
- [ ] Phase 01 — Better Auth core
- [ ] Phase 02 — User, account and session lifecycle
- [ ] Phase 03 — Organizations and hierarchy
- [ ] Phase 04 — Authorization and delegated administration
- [ ] Phase 05 — Audit and security events
- [ ] Phase 06 — Applications and service identities
- [ ] Phase 07 — OAuth/OIDC and federation
- [ ] Phase 08 — SIP identity and Digest authentication
- [ ] Phase 09 — ISP/RADIUS identity
- [ ] Phase 10 — Generic HA integration
- [ ] Phase 11 — Security hardening
- [ ] Phase 12 — Chaos, performance and production hardening
- [ ] Phase 13 — Production readiness
