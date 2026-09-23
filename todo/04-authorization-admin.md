# Phase 04 — Authorization and Delegated Administration

## Authorization

- [ ] Permission vocabulary
- [ ] Platform scope
- [ ] Organization scope
- [ ] Descendant scope
- [ ] Self-only scope
- [ ] Role resolution
- [ ] Target resolution
- [ ] Permission evaluation
- [ ] Explicit deny behavior

## Roles

- [ ] Platform superadmin
- [ ] Organization owner
- [ ] Organization admin
- [ ] Organization manager
- [ ] Member
- [ ] Custom organization roles where required

## Administration

- [ ] User management
- [ ] User creation
- [ ] User disable/ban
- [ ] Session management
- [ ] Impersonation
- [ ] Impersonation reason
- [ ] Impersonation expiry
- [ ] Privileged action checks

## Tests

- [ ] Parent admin can manage permitted descendant
- [ ] Parent cannot exceed configured scope
- [ ] Child cannot manage parent
- [ ] Sibling access denied
- [ ] Platform scope tested
- [ ] Impersonation audited

## Acceptance

- [ ] No authorization decision depends solely on a Better Auth role string
