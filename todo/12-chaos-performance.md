# Phase 12 — Chaos, Performance and Production Hardening

## Chaos

- [ ] Deterministic seed
- [ ] Node kill
- [ ] Node restart
- [ ] Leader kill
- [ ] Multiple-node failure
- [ ] Network delay
- [ ] Network drop
- [ ] Network duplicate
- [ ] Network reorder
- [ ] Network partition
- [ ] Partition healing
- [ ] Database unavailable
- [ ] Database failover
- [ ] Authentication during failure
- [ ] SIP authentication during failure
- [ ] Organization mutation during failure

## Correctness assertions

- [ ] No duplicate identity creation
- [ ] No privilege escalation
- [ ] No tenant breakout
- [ ] No stale authoritative node
- [ ] Safe recovery
- [ ] Audit continuity

## Performance

- [ ] Login latency
- [ ] Session latency
- [ ] Authorization latency
- [ ] Organization operation latency
- [ ] OIDC latency
- [ ] SIP authentication latency
- [ ] RADIUS latency
- [ ] HA failover duration

## Soak

- [ ] 1-hour soak
- [ ] 24-hour soak
- [ ] 72-hour soak
- [ ] Resource leak inspection

## Acceptance

- [ ] Reproducible chaos campaigns pass
- [ ] No known critical performance/resource issue remains
