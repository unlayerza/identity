# Unlayer Identity — Architecture, Roadmap & Engineering Contract

> The engineering contract for Unlayer Identity: one global identity/control plane powering web authentication, organizations, delegated administration, federation, SIP identity, ISP/service identity, and future authentication protocols.

This document is intentionally more detailed than a README. It defines the intended architecture, boundaries, security model, protocol adapters, HA expectations, testing strategy, roadmap, and definition of done.

## 1. Mission

Unlayer Identity is the central identity service for the Unlayer platform.

It should provide one coherent identity graph for:

- Unlayer users.
- Web accounts and sessions.
- Organizations and memberships.
- Hierarchical organizations.
- Applications and service identities.
- OAuth/OIDC federation.
- API credentials.
- SIP identities and SIP Digest authentication.
- Future ISP authentication such as RADIUS/PPPoE.
- Delegated administration.
- Platform administration.
- Audit and security events.

Better Auth is the authentication engine inside Unlayer Identity, not the complete definition of Unlayer Identity.

Better Auth already provides authentication, sessions, organizations/access control, plugins and extensibility; Unlayer adds the platform-wide identity graph, hierarchical organization model, protocol adapters, authorization scope and operational control plane. citeturn0search9turn0search1turn0search0

## 2. Core Principles

### One global identity plane

The default architecture is one global Better Auth instance serving the Unlayer identity system.

Do not create one Better Auth instance per customer as the normal architecture.

Enterprise isolation may eventually support dedicated identity instances, but this is an advanced deployment model rather than the default.

### One API surface

The canonical public service boundary is:

`api.unlayer.network`

Conceptually:

`/v1/identity`

Identity does not need to become a separate public API domain merely because it is a separate repository.

### Better Auth is the engine

Use Better Auth for its mature authentication/session/plugin machinery.

Build Unlayer-specific capabilities around it rather than replacing it unnecessarily.

### Protocol-specific authentication

The same human identity can have different protocol credentials:

```
user
├── web session
├── OAuth/OIDC identity
├── API credentials
├── SIP credentials
└── future ISP credentials
```

These credentials are related through the Unlayer identity graph but are not interchangeable.

### Authorization is separate from authentication

Authentication answers:

> Who are you?

Authorization answers:

> What may you do, where, and on whose behalf?

Unlayer's organization hierarchy and service scope belong to authorization.

## 3. Target Architecture

```
                         UNLAYER IDENTITY
                               |
                     +---------+---------+
                     |                   |
                 Better Auth       Unlayer Identity
                     |                   |
        +------------+------------+      +----------------+
        |            |            |      |                |
      users       sessions     accounts  org graph     audit
                                             |
                                      +------+------+
                                      |             |
                                  Org parent     database_id
                                      |
                                  descendants
```

The identity service is itself a platform service:

```
api.unlayer.network
└── /v1/identity
    ├── authentication
    ├── sessions
    ├── organizations
    ├── memberships
    ├── authorization
    ├── federation
    ├── credentials
    ├── SIP
    ├── ISP identity
    └── administration
```

## 4. Identity Data Model

The core user remains the canonical human identity.

Conceptual model:

```
User
 |
 +-- Account(s)
 +-- Session(s)
 +-- Organization Membership(s)
 +-- OAuth/OIDC identity
 +-- API credential(s)
 +-- SIP identity
 +-- ISP/service identity
```

The database must distinguish:

- human identity,
- authentication credential,
- session,
- organization membership,
- service identity,
- protocol credential,
- authorization grant,
- audit event.

Never collapse all of these into one generic credential table.

## 5. Better Auth Integration

Better Auth stores core authentication data such as users, sessions and accounts and allows plugins to add their own schema. Custom database adapters are supported through `createAdapterFactory`. citeturn0search2turn0search5

Initial implementation target:

- Better Auth.
- Native Bun server.
- Unlayer Database as the persistence layer.
- Custom Turso-compatible Better Auth adapter where required.
- Better Auth organization plugin.
- Unlayer hierarchy/control-plane tables beside Better Auth tables.
- Custom Unlayer plugins for protocol-specific identity capabilities.

Do not fork Better Auth unless a concrete requirement makes it unavoidable.

## 6. Organization Model

Better Auth organizations provide memberships, roles, permissions and optional teams. The built-in organization model is not sufficient by itself for Unlayer's hierarchical platform model, so Unlayer adds a parent/child organization graph. citeturn0search1

Conceptual table:

```
organization
id
parent_id
name
slug
type
database_id
status
plan
created_at
updated_at
```

Example:

```
Unlayer
├── Unlayer Identity
│   ├── Identity API
│   └── identity database
├── Unlayer Database
│   ├── Database API
│   └── database database
└── Acme Agency
    ├── Joe's Plumbing
    │   └── database: joe-db
    ├── Smith Attorneys
    │   └── database: smith-db
    └── Widget Corp
        └── database: widget-db
```

An organization may therefore own or manage child organizations.

## 7. Organization Authorization

Unlayer authorization must answer both:

1. Does the user have the required permission?
2. Is the target organization within the user's permitted scope?

Example conceptual roles:

```
platform_superadmin  global
organization_owner   self + descendants
organization_admin   self + descendants
organization_manager self
member               self/read
```

These names are Unlayer domain concepts. They do not have to map one-to-one to Better Auth's built-in role names.

Better Auth access control can provide the permission vocabulary; Unlayer adds hierarchical scope. Better Auth supports custom organization permissions and dynamic roles. citeturn0search1

Authorization flow:

```
authenticate
   ↓
identify user
   ↓
load membership
   ↓
resolve role
   ↓
resolve target organization
   ↓
resolve ancestor/descendant relationship
   ↓
evaluate permission
   ↓
allow / deny
```

A platform administrator must not receive unrestricted database or application access merely because they authenticated as an administrator.

## 8. Platform Administration

Better Auth's administrative capabilities should be treated as platform capabilities behind Unlayer authorization.

Potential capabilities:

- user management,
- create user,
- disable user,
- ban/unban,
- session management,
- impersonation,
- role management.

Any high-privilege action must be:

- explicitly authorized,
- scoped,
- audited,
- rate limited where appropriate,
- attributable to an actor.

Impersonation must never become anonymous support access.

Every impersonation should record:

- actor,
- target,
- organization scope,
- reason,
- request ID,
- start time,
- end time,
- originating IP,
- user agent,
- resulting session identity.

## 9. Audit System

Identity must have an immutable security/audit trail.

Conceptual model:

```
audit_event
id
actor_user_id
organization_id
target_organization_id
target_user_id
action
resource
resource_id
reason
ip_address
user_agent
metadata
created_at
```

Record at minimum:

- login,
- logout,
- failed authentication,
- password change,
- credential creation,
- credential revocation,
- MFA changes,
- organization creation,
- membership changes,
- role changes,
- organization hierarchy changes,
- administrator actions,
- impersonation,
- federation changes,
- SIP credential changes,
- ISP credential changes,
- security policy changes.

Audit events must not be editable through ordinary application APIs.

## 10. Sessions

Sessions are an identity security boundary.

Requirements:

- secure session identifiers,
- expiration,
- revocation,
- session listing,
- session termination,
- device/context metadata where appropriate,
- refresh/re-authentication policy,
- suspicious session detection hooks,
- audit events.

Different protocols may have different session semantics.

A SIP registration is not a web session.

A RADIUS authentication is not an OAuth session.

The identity graph connects them without pretending they are the same protocol.

## 11. Credentials

Credential categories should be explicitly modeled:

```
credential
├── password
├── passkey
├── OAuth/OIDC
├── API token
├── SIP Digest
└── ISP/RADIUS
```

Requirements:

- explicit credential type,
- owner identity,
- creation time,
- last-used time where applicable,
- expiration,
- revocation,
- rotation,
- audit trail,
- scope,
- metadata.

Secrets must never be logged.

## 12. OAuth / OIDC

Unlayer Identity should eventually operate as an identity provider and federation layer.

Capabilities:

- OAuth authorization.
- OIDC.
- application registration.
- client credentials.
- redirect URI validation.
- scopes.
- claims.
- consent.
- authorization codes.
- token lifecycle.
- signing-key management.
- key rotation.
- discovery metadata.
- JWKS.
- logout/session coordination.

External Unlayer applications should be able to authenticate against Unlayer Identity without creating a second user account.

Example:

```
User
  ↓
Unlayer Identity
  ↓
OIDC
  ↓
ISP / application / partner
```

The external application maps the stable identity subject to its local customer/account identifier.

## 13. Identity Federation

Federation should support:

- Unlayer as identity provider.
- Unlayer as relying party/client.
- social identity providers where appropriate.
- enterprise OIDC.
- future SAML/enterprise federation.

Federation configuration belongs to an organization/application scope, not globally by accident.

## 14. SIP Identity

SIP authentication is a major Unlayer Identity capability.

The architecture should be:

```
                  Unlayer Identity
                         |
                  +------+------+
                  |             |
             Better Auth    SIP adapter
                                |
                    +-----------+-----------+
                    |           |           |
                credentials   nonce      authorization
```

The SIP adapter must use the same canonical user/service identity graph while maintaining SIP-specific credentials.

SIP Digest should be implemented according to current SIP Digest standards. RFC 8760 adds SHA-256 and SHA-512/256 support and makes `qop` support required, updating the older MD5-centric SIP Digest behavior. citeturn0search4

Initial target:

- credential provisioning,
- realm management,
- nonce generation,
- nonce expiry,
- replay protection,
- challenge generation,
- digest verification,
- algorithm negotiation,
- qop handling,
- registration authorization,
- call authorization,
- credential rotation,
- revocation,
- audit.

Never store reusable plaintext SIP passwords.

The SIP adapter may be implemented as a Better Auth plugin because Better Auth plugins can add endpoints, schemas, middleware, hooks and rate-limit rules. citeturn0search0

## 15. SIP Authorization

Authentication is not enough.

A successfully authenticated SIP identity must still be authorized to:

- register a device,
- use a number,
- originate a call,
- terminate a call,
- use a trunk,
- access a tenant,
- use a service,
- consume allocated resources.

Example:

```
SIP credential
   ↓
canonical identity
   ↓
organization
   ↓
service
   ↓
number / trunk / endpoint
   ↓
authorization
```

## 16. ISP / RADIUS Identity

Future Unlayer Identity should be able to act as an identity source for ISP infrastructure.

Potential protocols:

- RADIUS.
- PPPoE authentication.
- network access control.
- service activation.
- subscriber policy lookup.

Conceptually:

```
subscriber
   ↓
Unlayer Identity
   ├── web identity
   ├── ISP account
   ├── SIP identity
   └── network credentials
```

The RADIUS protocol adapter must not pretend that RADIUS is a web authentication mechanism.

Instead it maps a network authentication request onto the canonical identity/service model.

Future capabilities:

- Access-Request handling.
- username/password authentication.
- service authorization.
- subscriber attributes.
- rate/profile selection.
- accounting integration.
- session state.
- service suspension/protected-service state.
- reseller/ISP delegation.

## 17. Protected Service State

Unlayer may eventually support a service state where an ISP subscriber's identity remains active while network service is placed into a controlled state.

Possible states:

```
active
restricted
protected
suspended
terminated
```

This must be implemented as a service-policy concept rather than an authentication hack.

Identity remains identifiable while network/service authorization changes.

## 18. Resellers

Identity must support delegated reseller administration.

Example:

```
Unlayer
└── Reseller A
    ├── Customer A
    ├── Customer B
    └── Customer C
```

A reseller administrator should be able to manage only the organizations/users/services within its authorized scope.

Resellers must not receive platform-global privileges.

## 19. Applications and Service Identities

Not every identity is a human.

Support:

- application identities,
- service accounts,
- API clients,
- machine credentials,
- infrastructure nodes.

Human users and machine identities must remain distinguishable.

Example:

```
organization
├── users
├── applications
├── service accounts
└── devices
```

## 20. API Credentials

Support scoped credentials for automation.

Requirements:

- unique identifier,
- secret/token,
- scope,
- owner,
- organization,
- expiration,
- rotation,
- revocation,
- last-used metadata,
- audit.

Never expose the full secret after initial creation unless the credential protocol explicitly requires it.

## 21. WebAuthn / Passkeys / MFA

Identity should support strong modern authentication.

Roadmap capabilities:

- passkeys/WebAuthn,
- TOTP,
- recovery codes,
- trusted-device policy,
- step-up authentication,
- MFA enforcement by organization,
- MFA enforcement by application,
- administrative MFA.

Better Auth already has plugin-based extensibility for authentication features, so these should be integrated rather than reinvented where suitable. citeturn0search11

## 22. Hooks and Plugins

Use Better Auth hooks for lifecycle customization where a hook is sufficient. Use a custom plugin when Unlayer needs a durable feature surface, schemas, endpoints, middleware or client APIs. citeturn0search10turn0search0

Candidate Unlayer plugins:

- hierarchical organization plugin,
- audit plugin,
- SIP plugin,
- service identity plugin,
- ISP/RADIUS plugin,
- platform administration plugin,
- API credential plugin,
- federation plugin.

## 23. Database Architecture

Identity is itself a consumer of Unlayer Database.

Target:

```
Unlayer Identity
       |
       v
Unlayer Database
       |
       +-- identity data
       +-- sessions
       +-- organizations
       +-- memberships
       +-- credentials
       +-- audit
```

The Identity service should not own a bespoke persistence architecture if the Unlayer Database service provides the required durability and HA.

## 24. HA Requirements

Identity must eventually run on multiple Bun server processes/nodes using the generic Unlayer HA subsystem.

Desired topology:

```
                 API / Edge
                     |
          +----------+----------+
          |          |          |
       Identity A Identity B Identity C
          |          |          |
          +----------+----------+
                     |
              Unlayer Database
```

Identity application servers should be as stateless as practical.

Persistent state belongs in Unlayer Database.

Node-local state must be disposable.

## 25. Generic HA Reuse

Identity must consume the generic HA capability developed in `unlayerza/database`.

Do not copy database-specific leader/follower code into Identity.

Identity should depend on generic capabilities such as:

- node identity,
- health,
- membership,
- leader election,
- fencing,
- quorum,
- failover,
- recovery,
- chaos hooks.

This repository should prove that the HA subsystem can operate an application service as well as a database cluster.

## 26. Security Model

Required from the beginning:

- secure secrets,
- strong authentication,
- authorization,
- tenant isolation,
- CSRF protection where applicable,
- trusted-origin validation,
- rate limiting,
- brute-force protection,
- session security,
- credential rotation,
- audit logging,
- replay protection,
- secure cookies,
- TLS in production,
- secure headers,
- request correlation,
- secret redaction.

Better Auth supports trusted origins, rate limiting, hooks and plugin middleware that can participate in these controls. citeturn0search0turn0search10

## 27. Threat Model

Explicitly model:

- credential stuffing,
- password spraying,
- stolen sessions,
- token theft,
- replay attacks,
- nonce reuse,
- compromised SIP credentials,
- malicious reseller,
- compromised application,
- privilege escalation,
- organization traversal,
- tenant breakout,
- forged service identity,
- compromised node,
- compromised control-plane credential,
- malicious administrator,
- backup compromise.

Every threat must have a documented mitigation or explicit residual-risk decision.

## 28. Auditability

Identity security operations must be observable.

Events should include:

- authentication success/failure,
- session creation/revocation,
- credential creation/revocation,
- organization creation,
- membership change,
- role change,
- hierarchy change,
- admin operation,
- impersonation,
- federation configuration,
- SIP authentication,
- SIP credential changes,
- network authentication,
- policy changes.

Audit events must include enough information to reconstruct what happened without storing secrets.

## 29. Testing Strategy

Testing is a core feature.

Required levels:

1. Unit tests.
2. Integration tests.
3. Real Better Auth tests.
4. Real HTTP tests.
5. Multi-process HA tests.
6. Protocol tests.
7. Security tests.
8. Federation tests.
9. Failure-injection tests.
10. Chaos tests.
11. Performance tests.
12. Soak tests.

## 30. Local HA Harness

Identity should run multiple actual Bun processes locally:

```
identity-a -> 7201
identity-b -> 7202
identity-c -> 7203
```

The test harness must support:

- start,
- stop,
- hard kill,
- restart,
- delayed messages,
- dropped messages,
- duplicated messages,
- reordered messages,
- network partition,
- partition healing.

The same HA harness used by Database should eventually be reusable here.

## 31. Identity Chaos Mode

Chaos tests must cover:

- leader process death,
- follower process death,
- simultaneous node death,
- network partition,
- delayed control-plane requests,
- duplicated requests,
- stale node return,
- database unavailable,
- database failover during authentication,
- database failover during session creation,
- database failover during SIP authentication,
- database failover during organization changes.

Correctness requirements:

- no duplicate identities,
- no privilege escalation,
- no cross-tenant access,
- no split-brain administrative state,
- safe recovery after node return.

## 32. Protocol Test Suites

### Web

- signup,
- signin,
- signout,
- session,
- password change,
- credential revocation,
- MFA,
- passkey.

### Organization

- create organization,
- nested organization,
- membership,
- role,
- descendant authorization,
- sibling denial,
- parent authorization,
- reseller boundaries.

### OIDC

- discovery,
- authorization,
- code exchange,
- token validation,
- claims,
- JWKS,
- key rotation,
- logout,
- invalid redirect URI.

### SIP

- challenge,
- valid digest,
- invalid digest,
- stale nonce,
- nonce replay,
- qop,
- SHA-256,
- SHA-512/256,
- authorization,
- revocation.

### RADIUS

- Access-Request,
- valid credentials,
- invalid credentials,
- policy response,
- service restriction,
- accounting,
- reseller scope.

## 33. Correctness Invariants

1. One canonical user identity must not silently become multiple unrelated users.
2. A credential belongs to exactly one identity.
3. Authentication never implies unrestricted authorization.
4. A child organization cannot access a sibling organization.
5. A parent administrator can only administer descendants where its role permits it.
6. Platform privileges are distinct from organization privileges.
7. Revoked credentials cannot authenticate.
8. Expired credentials cannot authenticate.
9. SIP nonce replay cannot succeed.
10. Stale HA nodes cannot perform authoritative administrative writes.
11. Audit records cannot be silently rewritten.
12. Secrets never appear in logs.
13. A compromised tenant cannot traverse into another tenant.
14. Machine identities cannot accidentally inherit human administrative privileges.

## 34. API Shape

Canonical service boundary:

```
api.unlayer.network
└── /v1/identity
    ├── /auth
    ├── /users
    ├── /sessions
    ├── /organizations
    ├── /memberships
    ├── /roles
    ├── /credentials
    ├── /applications
    ├── /federation
    ├── /oidc
    ├── /sip
    ├── /isp
    └── /audit
```

Better Auth's native endpoints may remain internally exposed through its handler/API surface, while Unlayer's stable API contract wraps or composes them where platform semantics require it. Better Auth exposes an API object for its endpoints, including plugin endpoints. citeturn0search7

Do not unnecessarily duplicate every Better Auth endpoint merely to make it "Unlayer."

## 35. Service-to-Service Identity

Future Unlayer services should authenticate to Identity using explicit service credentials.

Examples:

```
Database -> Identity
Voice -> Identity
API -> Identity
Dashboard -> Identity
Hosting -> Identity
```

Service authentication must support:

- service identity,
- credential rotation,
- scoped permissions,
- audience,
- expiration,
- audit.

## 36. Observability

Structured events:

- identity_started,
- authentication_success,
- authentication_failed,
- session_created,
- session_revoked,
- credential_created,
- credential_revoked,
- organization_created,
- membership_changed,
- role_changed,
- authorization_denied,
- impersonation_started,
- impersonation_ended,
- federation_changed,
- sip_authentication,
- radius_authentication,
- leader_elected,
- leader_lost,
- node_joined,
- node_left.

Metrics:

- authentication latency,
- authentication failures,
- session creation latency,
- authorization latency,
- organization operations,
- credential operations,
- SIP authentication latency,
- RADIUS authentication latency,
- federation latency,
- HA failover duration,
- database latency,
- rate-limit events.

Never emit passwords, tokens, SIP secrets, private keys or equivalent sensitive material.

## 37. Operational Requirements

Support:

- graceful shutdown,
- startup health,
- readiness checks,
- liveness checks,
- rolling deployment,
- node drain,
- credential rotation,
- configuration reload where safe,
- database migration,
- audit export,
- emergency access procedures.

## 38. Recovery

Identity recovery must include:

- database restore,
- PITR through Unlayer Database,
- application node replacement,
- HA failover,
- credential recovery procedures,
- signing-key recovery,
- federation configuration recovery,
- audit preservation.

Identity must not rely on node-local state for recovery.

## 39. Production Readiness

Minimum meaningful acceptance:

```
create user
authenticate
create organization
create child organization
assign membership
authorize parent administration
deny sibling access
create application
authenticate application
issue OIDC identity
authenticate SIP identity
revoke SIP credential
perform admin operation
audit every privileged action
kill identity leader
fail over
continue authentication
restore failed node
verify convergence
restore database
verify identity state
```

## 40. Non-Goals

The first version is not intended to:

- replace every enterprise IAM product,
- implement every authentication protocol immediately,
- become the database engine,
- become the telecom core,
- embed SIP media processing,
- embed RADIUS accounting storage unnecessarily,
- create separate identity instances for every tenant.

## 41. Open Engineering Questions

These must be resolved through current documentation and experiments:

- Exact Better Auth version selected.
- Exact Better Auth plugin configuration.
- Exact custom adapter behavior against Unlayer Database.
- Organization hierarchy representation.
- Authorization scope algorithm.
- Platform administrator model.
- Impersonation policy.
- OIDC provider/client boundaries.
- Signing key storage and rotation.
- SIP credential representation.
- SIP HA semantics.
- SIP nonce storage/replication requirements.
- RADIUS implementation boundary.
- Service identity format.
- API token format.
- Secret rotation protocol.
- Audit retention.
- Privacy/data retention requirements.
- Regional identity requirements.
- Data residency.
- Disaster recovery RPO/RTO.
- Generic HA integration boundary.

## 42. Development Order

1. Project foundation.
2. Better Auth core.
3. Unlayer Database integration.
4. User/session lifecycle.
5. Organization integration.
6. Hierarchical organization graph.
7. Authorization scope.
8. Audit.
9. Platform administration.
10. API/service identities.
11. OAuth/OIDC.
12. SIP identity.
13. ISP/RADIUS identity.
14. Generic HA integration.
15. Chaos testing.
16. Security hardening.
17. Performance/soak.
18. Production readiness.

## 43. First Milestone

The first meaningful milestone is deliberately small:

```
Bun server
+
Better Auth
+
Unlayer Database
+
signup
+
signin
+
session
+
organization
+
audit
```

Then prove:

```
identity-a
identity-b
identity-c

kill identity leader
fail over
authenticate
continue normal operation
restart old node
resynchronize
```

Then expand into federation and protocol identity.

## 44. Relationship to Unlayer Database

Database provides:

- durable state,
- replication,
- quorum,
- snapshots,
- backups,
- PITR,
- recovery.

Identity provides:

- human identity,
- credentials,
- authentication,
- authorization,
- organizations,
- protocol identity,
- audit.

Neither should absorb the other's responsibilities.

## 45. Definition of Done

Identity is production-ready only when:

- authentication works,
- authorization is hierarchical and tested,
- organizations are isolated,
- privileged operations are audited,
- credentials are safely managed,
- federation is tested,
- SIP authentication is tested,
- future ISP identity boundaries are defined,
- HA failover is demonstrated,
- chaos campaigns pass,
- backups/PITR have been exercised through Database,
- observability is sufficient for incident response,
- security review findings are resolved or explicitly accepted,
- operational recovery has been rehearsed.

---

## Reference material

Current official documentation used to establish the architecture:

- Better Auth introduction: https://better-auth.com/docs/introduction
- Better Auth database: https://better-auth.com/docs/concepts/database
- Better Auth plugins: https://better-auth.com/docs/concepts/plugins
- Better Auth organization: https://better-auth.com/docs/plugins/organization
- Better Auth custom database adapter: https://better-auth.com/docs/guides/create-a-db-adapter
- Better Auth API: https://better-auth.com/docs/concepts/api
- Better Auth hooks: https://better-auth.com/docs/concepts/hooks
- RFC 8760 — SIP Digest Access Authentication: https://www.rfc-editor.org/rfc/rfc8760.html
