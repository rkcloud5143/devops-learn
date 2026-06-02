# SRE Concepts — SLI, SLO, SLA & Error Budgets 🎯

How to measure and manage reliability like Google.

---

## The Core Idea

```
"100% reliability is the wrong target."
  — Google SRE Book

If your service is 100% reliable, you're moving too slowly.
Error budgets let you balance reliability with speed of innovation.
```

---

## SLI, SLO, SLA — What's the Difference?

```
SLI (Service Level Indicator)
  = What you MEASURE
  "99.2% of requests completed in < 200ms"

SLO (Service Level Objective)
  = What you TARGET (internal)
  "We aim for 99.9% availability"

SLA (Service Level Agreement)
  = What you PROMISE (external, with consequences)
  "We guarantee 99.5% uptime or you get credits"

SLI ──measures──▶ SLO ──promises──▶ SLA

SLO is always stricter than SLA.
If SLA = 99.5%, set SLO = 99.9% so you have buffer.
```

---

## Common SLIs

| SLI | What it measures | Example |
|-----|-----------------|---------|
| **Availability** | Is the service up? | Successful requests / total requests |
| **Latency** | How fast? | 95th percentile response time < 200ms |
| **Error rate** | How often does it fail? | Errors / total requests < 0.1% |
| **Throughput** | How much? | Requests per second |
| **Durability** | Is data safe? | Data loss events per year |
| **Freshness** | How stale? | Data updated within 1 minute |

### Measuring SLIs
```
Availability = (total requests - error requests) / total requests × 100

Example:
  1,000,000 requests, 500 errors
  (1,000,000 - 500) / 1,000,000 = 99.95%
```

---

## The Nines Table

```
Availability    Downtime/year    Downtime/month    Downtime/week
──────────────  ──────────────   ──────────────    ─────────────
99%             3.65 days        7.3 hours         1.68 hours
99.9%           8.77 hours       43.8 minutes      10.1 minutes
99.95%          4.38 hours       21.9 minutes      5.04 minutes
99.99%          52.6 minutes     4.38 minutes      1.01 minutes
99.999%         5.26 minutes     26.3 seconds      6.05 seconds
```

Going from 99.9% to 99.99% is **10x harder and 10x more expensive**.

---

## Error Budgets

```
Error Budget = 100% - SLO

If SLO = 99.9% availability:
  Error budget = 0.1% = 43.8 minutes of downtime per month

You can "spend" this budget on:
  ✅ Deploying new features (risk of bugs)
  ✅ Infrastructure changes
  ✅ Experiments
  ✅ Planned maintenance

If budget is exhausted:
  🛑 STOP deploying new features
  🛑 Focus on reliability
  🛑 Fix tech debt
  🛑 Improve monitoring
```

### Error Budget Policy
```
Budget remaining > 50%  → Ship features freely
Budget remaining 20-50% → Ship with extra caution, more testing
Budget remaining < 20%  → Only reliability work, no new features
Budget exhausted (0%)   → Feature freeze until budget recovers
```

---

## Toil

```
Toil = manual, repetitive, automatable work that scales with service size.

Examples of toil:
  ❌ Manually restarting pods every morning
  ❌ Manually rotating certificates
  ❌ Manually scaling servers for traffic spikes
  ❌ Copy-pasting deployment commands
  ❌ Manually checking dashboards

NOT toil:
  ✅ Writing automation to fix the above
  ✅ Architecture design
  ✅ Code reviews
  ✅ Incident response (novel problems)

Google's rule: SREs should spend < 50% time on toil.
The rest goes to engineering work that reduces future toil.
```

---

## Incident Management

```
Severity Levels:
  SEV1 (Critical)  — Service down, all users affected
  SEV2 (Major)     — Significant degradation, many users affected
  SEV3 (Minor)     — Partial issue, some users affected
  SEV4 (Low)       — Cosmetic, workaround exists

Incident Roles:
  Incident Commander (IC) — Coordinates response
  Operations Lead         — Does the technical work
  Communications Lead     — Updates stakeholders
  Scribe                  — Documents timeline

Incident Lifecycle:
  Detect → Triage → Mitigate → Resolve → Post-mortem
```

---

## Post-Mortems (Blameless)

```
Template:
  1. Summary        — What happened in 2 sentences
  2. Impact         — Users affected, duration, revenue impact
  3. Timeline       — Minute-by-minute what happened
  4. Root Cause     — 5 Whys analysis
  5. What went well — Detection, response
  6. What went wrong — Gaps
  7. Action Items   — Specific, assigned, deadlined
  8. Lessons Learned — What we'll do differently

Key principle: BLAMELESS
  Focus on systems and processes, not people.
  "The deploy script didn't have rollback" not "Bob broke production"
```

---

## MTTR, MTTD, MTTF, MTBF

```
MTTD (Mean Time To Detect)    — How fast you notice the problem
MTTR (Mean Time To Recover)   — How fast you fix it
MTTF (Mean Time To Failure)   — How long until next failure
MTBF (Mean Time Between Failures) = MTTF + MTTR

Timeline:
  ──[Working]──│──[Down]──│──[Working]──│──[Down]──│──[Working]──
               ↑          ↑             ↑          ↑
            Failure    Recovery      Failure    Recovery
               │←─MTTR──→│             │←─MTTR──→│
               │←──────── MTBF ────────→│

Goal: Reduce MTTD and MTTR. Increase MTTF.
```

---

## SRE vs DevOps

```
DevOps = Culture and practices (CI/CD, automation, collaboration)
SRE    = Implementation of DevOps with specific practices (SLOs, error budgets, toil reduction)

"SRE is what happens when you ask a software engineer to design an operations team."
  — Ben Treynor, Google

DevOps says: "Break down silos between dev and ops"
SRE says:    "Here's exactly how, with measurable targets"
```

---

*SRE concepts show up in every senior DevOps interview. Know them cold! 🎯*
