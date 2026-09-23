# Phase 08 — SIP Identity and Digest Authentication

## Identity

- [ ] SIP identity model
- [ ] SIP username
- [ ] SIP realm
- [ ] Credential lifecycle
- [ ] Number/service association
- [ ] Trunk association
- [ ] Credential rotation
- [ ] Revocation

## Digest

- [ ] Challenge generation
- [ ] Nonce generation
- [ ] Nonce expiry
- [ ] Replay protection
- [ ] qop
- [ ] SHA-256
- [ ] SHA-512/256
- [ ] Algorithm policy
- [ ] Digest verification
- [ ] Invalid digest rejection
- [ ] Stale nonce handling

## Authorization

- [ ] Registration authorization
- [ ] Number authorization
- [ ] Origination authorization
- [ ] Termination authorization
- [ ] Trunk authorization
- [ ] Organization authorization

## Tests

- [ ] Valid digest
- [ ] Invalid digest
- [ ] Replay
- [ ] Expired nonce
- [ ] Wrong realm
- [ ] Wrong user
- [ ] qop
- [ ] SHA-256
- [ ] SHA-512/256
- [ ] Revoked credential

## Acceptance

- [ ] Real SIP Digest client/server test vectors pass
- [ ] SIP credentials remain integrated with canonical Unlayer identities
