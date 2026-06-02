# Advanced Interview Q&A — SRE, DevOps, Platform Engineer & DevSecOps 🎯

Role-specific questions that differentiate senior candidates. These go beyond generic "what is X?" questions.

---

# SRE (Site Reliability Engineer)

## Reliability & SLOs

**Q: How do you choose SLIs for a service you've never seen before?**
A: Start with what users care about:
- API service → Availability (success rate), Latency (p99), Error rate
- Data pipeline → Freshness (how stale), Correctness, Throughput
- Storage → Durability, Availability, Latency

Method: Look at the service from the user's perspective. "What would they complain about?" That's your SLI.

**Q: Your SLO is 99.9% but you're currently at 99.95%. Should you be happy?**
A: Not necessarily. Questions to ask:
- Is the remaining error budget being used for deploying features? (Good — we're innovating)
- Is it sitting unused? (Bad — we're being too cautious, moving too slowly)
- Are we one bad deploy away from blowing the budget? (Risky — need safeguards)

The goal isn't to maximize reliability — it's to use error budget intentionally.

**Q: How do you handle a team that keeps burning their error budget?**
A:
1. First: Are the SLOs correct? Maybe they're too aggressive.
2. If SLOs are right: Implement error budget policy
   - Budget < 20% → reliability-only work, no new features
   - Budget exhausted → feature freeze, post-mortem required
3. Identify patterns: Is it deploys? Infrastructure? Dependencies?
4. Invest in: better testing, canary deploys, automated rollback

**Q: Explain the difference between availability and reliability.**
A:
- Availability = Is it up? (binary — can I reach it)
- Reliability = Does it work correctly? (includes correctness, latency, consistency)

A service can be "available" (responds to requests) but "unreliable" (returns wrong data, is very slow, or is inconsistent).

---

## Incident Management

**Q: You're the on-call SRE. You get paged at 3am. Walk me through your first 5 minutes.**
A:
1. **Acknowledge alert** (stop escalation)
2. **Read the alert** — what metric breached? Which service? Since when?
3. **Check dashboard** — is it getting worse, stable, or recovering?
4. **Assess blast radius** — all users? One region? One customer?
5. **Decide: mitigate now or investigate?**
   - If actively degrading → mitigate first (rollback, scale, failover)
   - If stable → investigate root cause

I'm NOT opening my laptop to write code at 3am. I'm looking for the fastest path to restore service.

**Q: How do you run a blameless post-mortem?**
A:
- Focus on systems and processes, never individuals
- "The deploy pipeline allowed untested code through" not "Bob pushed bad code"
- Ask: "What can we change so this class of failure can't happen again?"
- Every action item must be: specific, assigned, deadlined
- Share widely — failures are learning opportunities for the whole org
- Follow up: Track action items to completion (most post-mortems fail here)

**Q: How do you reduce alert fatigue?**
A:
- Every alert must be **actionable** — if there's nothing to do, delete it
- Alert on **symptoms**, not causes (users can't log in > CPU is high)
- **Tier alerts**: page-worthy (SEV1) vs Slack (SEV3) vs log-only
- **Aggregate**: 100 pods crashing = 1 alert, not 100
- **Auto-resolve**: if condition clears, close the alert
- **Review quarterly**: delete alerts nobody acts on
- **Measure**: track page frequency per engineer per week (target: < 2)

---

## Capacity & Performance

**Q: How do you do capacity planning for a service you own?**
A:
1. **Measure current**: requests/sec, CPU/memory utilization at peak
2. **Model growth**: traffic patterns, business growth projections
3. **Find ceiling**: load test to find breaking point
4. **Set headroom**: plan for 2x peak (handle spikes + one AZ failure)
5. **Automate**: Auto Scaling for organic growth, manual for step changes
6. **Review quarterly**: compare projection vs actual

Formula: `Required capacity = (Peak load × Growth factor) / (1 - Failure domain)`
If one AZ can fail: capacity = peak × growth / 0.67 (need to survive losing 1/3)

**Q: How would you design a system to handle 10x traffic spike (Black Friday)?**
A:
- **Pre-scale** infrastructure days before (don't rely on auto-scaling alone)
- **Load test** at 10x to find bottlenecks BEFORE the event
- **Circuit breakers** for non-critical paths (recommendations, analytics)
- **Feature flags** to shed load (disable expensive features)
- **Caching** aggressively (even stale data is better than errors)
- **Queue work** — accept requests fast, process async
- **War room** — engineers on standby during the event
- **Runbooks** — pre-written "if X happens, do Y"

---

## Toil & Automation

**Q: Give me an example of toil you automated away.**
A: (Use your real experience) Example:
"Patch management was manual — vendor ran it, they left, no docs. I built Terraform modules for SSM Patch Manager with scheduled maintenance windows, pre-patch AMI backups, a Lambda for compliance reporting to Slack, and Jira integration for change tickets. Went from 2 days of manual work monthly to fully automated with self-service reporting."

**Q: How do you decide what to automate vs leave manual?**
A: Automate if:
- It's done > 2x per week
- It's error-prone when done manually
- It scales with service growth
- It blocks other people

Keep manual if:
- It's a one-time task
- It requires judgment/creativity
- Automating costs more than the toil over 6 months
- It's genuinely rare (< 1x per quarter)

---

# DevOps Engineer (Senior/Staff)

## Architecture & Design

**Q: How would you design a CI/CD pipeline for 50+ microservices?**
A:
- **Shared workflows** — Central repo with reusable pipelines (workflow_call)
- **Mono-repo or multi-repo** — affects trigger strategy
- **Path filters** — only build services that changed
- **Artifact promotion** — same image dev → staging → prod (never rebuild)
- **GitOps** — CI pushes image tag, ArgoCD deploys
- **Automated quality gates** — tests, security scan, compliance check
- **Progressive delivery** — canary → percentage rollout → full
- **Self-service** — teams onboard by adding 5-line workflow file

**Q: How do you handle database schema changes in a zero-downtime deployment?**
A: Expand-and-contract pattern:
1. **Expand**: Add new column (nullable, with default) — backward compatible
2. **Migrate**: Backfill data from old to new column
3. **Deploy**: New code uses new column
4. **Contract**: Remove old column (in a later release)

Never in zero-downtime:
- Rename columns directly
- Change column types
- Add NOT NULL without default
- Drop columns that old code still reads

**Q: You inherit a legacy monolith. How do you start modernizing?**
A:
1. **Don't rewrite** — strangler fig pattern instead
2. **Add observability first** — you can't improve what you can't see
3. **Identify boundaries** — find loosely coupled components
4. **Extract one service** — smallest, lowest risk first
5. **API gateway** — route traffic to monolith OR new service
6. **Prove the pattern** — then repeat for next component
7. **Data** — hardest part. Start with read replicas, eventually separate DBs

**Q: How do you manage secrets across 50 services in multiple environments?**
A:
- **Centralized store**: AWS Secrets Manager or Vault
- **No secrets in Git** (even encrypted, unless using SOPS for GitOps)
- **OIDC/IAM roles** — services authenticate without credentials where possible
- **Rotation**: Automated for DB passwords, API keys
- **Least privilege**: Each service only accesses its own secrets
- **Audit**: CloudTrail logs every secret access
- **Emergency rotation**: Documented runbook, tested quarterly

---

## Scaling & Reliability

**Q: Your service needs to be multi-region active-active. What are the challenges?**
A:
- **Data consistency** — eventual consistency is usually required (CAP theorem)
- **Conflict resolution** — last-writer-wins, CRDTs, or application logic
- **Data replication lag** — users might see stale data
- **DNS routing** — latency-based + health checks
- **Session handling** — sessions can't be local (use DynamoDB/Redis global)
- **Deployment coordination** — deploy to one region first, verify, then others
- **Cost** — 2x infrastructure, cross-region data transfer charges
- **Testing** — how do you test failover without impacting users?

**Q: How do you implement infrastructure drift detection and remediation?**
A:
- **Detection**: `terraform plan` in CI on schedule (daily) — any diff = drift
- **ArgoCD self-heal** — automatically reverts manual K8s changes
- **AWS Config rules** — detect non-compliant resources
- **Alerting**: Drift detected → Slack notification → engineer investigates
- **Prevention**: Restrict console access, require IaC for all changes
- **Remediation**: Auto-apply for low-risk (tags), manual review for high-risk (security groups)

---

## Culture & Process

**Q: A developer says "DevOps is slowing us down." How do you respond?**
A:
1. **Listen** — understand their specific pain point
2. **Measure** — what's their lead time from commit to production?
3. **Common issues**:
   - Pipeline too slow → parallelize, cache, optimize
   - Too many approvals → automate what can be automated
   - Environment issues → self-service environments
   - "It works on my machine" → better local dev experience
4. **The goal is shared** — we both want fast, safe delivery
5. **Fix ONE thing** — don't overhaul, show quick improvement

**Q: How do you handle a team that keeps making the same mistakes?**
A:
- **Make the right thing easy** — paved paths, golden templates
- **Make the wrong thing hard** — policy-as-code, guardrails
- **Don't rely on documentation** — nobody reads it
- **Automate the check** — pre-commit hooks, CI gates
- **Blameless** — focus on systemic issues, not people
- Example: Teams keep forgetting encryption → default policy encrypts everything

---

# Platform Engineer

## Internal Developer Platform

**Q: What is a platform team's job?**
A: Reduce cognitive load on product teams. They should think about their code, not infrastructure.

Provide:
- Self-service infrastructure (no tickets for a new environment)
- Golden paths (opinionated defaults that work out of the box)
- Guardrails (prevent mistakes without blocking productivity)
- Observability (built-in monitoring, logging, tracing)
- Documentation (that's actually useful)

**Q: How do you design a self-service developer platform?**
A:
```
Developer experience:
  "I need a new service" → Fill template → PR → Auto-provisioned

Behind the scenes:
  Template → GitHub repo scaffolded
           → CI/CD pipeline attached
           → K8s namespace created
           → Secrets store configured
           → Monitoring dashboards created
           → DNS entry provisioned

Tools: Backstage, Port, Humanitec, or custom (ArgoCD + Crossplane + templates)
```

**Q: How do you decide between building vs buying platform tools?**
A:
Build if:
- Your workflow is truly unique
- Available tools need heavy customization
- You have the team to maintain it long-term
- It's a core differentiator

Buy/adopt if:
- Standard problem (CI/CD, monitoring, logging)
- Team is small (< 5 platform engineers)
- Speed to value matters
- Community/vendor handles upgrades and security

**Q: How do you measure platform team success?**
A: Developer productivity metrics:
- **Lead time** — commit to production (target: < 1 hour)
- **Deployment frequency** — how often teams deploy (target: daily)
- **Onboarding time** — new service from zero to production (target: < 1 day)
- **Self-service ratio** — % of requests handled without platform team involvement
- **Developer satisfaction** — survey (NPS score for internal platform)
- **MTTR** — mean time to recovery (shows if observability works)

**Q: How do you handle platform adoption when teams resist?**
A:
- **Don't mandate** — make it so good they want to use it
- **Start with early adopters** — friendly team, prove value
- **Reduce their pain** — what's their #1 complaint? Solve that first
- **Paved path != only path** — allow escape hatches
- **Show metrics** — "Team A deploys 5x faster since adopting the platform"
- **Embed engineers** — temporarily join a product team to understand their needs

---

## Multi-Tenancy & Isolation

**Q: How do you provide isolated environments for 20 product teams on shared infrastructure?**
A:
- **Kubernetes namespaces** — per team/service
- **Resource quotas** — prevent one team from consuming all resources
- **Network policies** — isolate team traffic
- **RBAC** — teams can only see/modify their own resources
- **Separate node pools** — for strict isolation (regulated workloads)
- **Cost allocation** — tag everything, show per-team spend

**Q: How do you manage Kubernetes at scale (10+ clusters)?**
A:
- **Fleet management** — Cluster API, EKS Blueprints, or Crossplane
- **GitOps** — ArgoCD ApplicationSets deploy across all clusters
- **Consistent tooling** — same add-ons (monitoring, ingress, secrets) everywhere
- **Cluster lifecycle** — automated upgrades, node rotation
- **Central observability** — all clusters feed into one monitoring stack
- **Policy enforcement** — OPA/Kyverno across all clusters

---

# DevSecOps

## Shift-Left Security

**Q: How do you implement "shift-left" security without blocking developers?**
A:
- **Phase 1 (Inform)**: Run scans, show results, don't break builds
- **Phase 2 (Soft fail)**: Warn on critical findings, allow override with justification
- **Phase 3 (Hard fail)**: Block CRITICAL only, allow HIGH with tech debt ticket
- **Never**: Block everything on day one (you'll get ignored/bypassed)

Key principle: Security should be a speed bump, not a roadblock.

**Q: Design a secure CI/CD pipeline from scratch.**
A:
```
Pre-commit:
  • git-secrets / trufflehog (prevent secret commits)
  • Pre-commit hooks (linting, formatting)

CI (on every PR):
  • SAST — SonarQube, Semgrep (code vulnerabilities)
  • SCA — Dependabot, Snyk (dependency vulnerabilities)  
  • IaC scanning — tfsec, checkov (Terraform misconfigurations)
  • Container scanning — Trivy (image vulnerabilities)
  • Secret scanning — TruffleHog (leaked secrets in history)
  • License compliance — check OSS licenses

CD (on deploy):
  • Image signing — Cosign (verify image integrity)
  • Admission control — OPA/Kyverno (enforce policies in K8s)
  • Runtime protection — Falco (detect anomalous behavior)

Post-deploy:
  • DAST — OWASP ZAP (test running application)
  • Penetration testing — scheduled quarterly
  • Bug bounty — continuous
```

**Q: A developer says a vulnerability is a false positive. How do you handle it?**
A:
1. **Verify** — is it actually exploitable in your context?
2. **Document** — if false positive, add to ignore list WITH explanation
3. **Don't just suppress** — create an exception record:
   - Who approved it
   - Why it's false positive
   - When to re-evaluate (expiration date)
4. **Automate** — maintain a `.trivyignore` / `.snyk` policy file in the repo
5. **Audit** — review exceptions quarterly

**Q: How do you handle a supply chain attack (compromised dependency)?**
A:
1. **Detect**: Dependabot/Snyk alerts, CISA advisories
2. **Assess**: Is the compromised version in our code? In production?
3. **Contain**: Pin known-good version, block vulnerable version in registry
4. **Remediate**: Update or remove dependency
5. **Prevent**:
   - Lock files (package-lock.json, go.sum)
   - Private registry mirror (Artifactory, CodeArtifact)
   - Allow-list approved packages
   - SBOM generation for all artifacts
   - Reproducible builds

---

## Container & Kubernetes Security

**Q: How do you secure a Kubernetes cluster?**
A:
```
Control Plane:
  • RBAC (least privilege, no cluster-admin for developers)
  • API audit logging enabled
  • OIDC authentication (no long-lived tokens)
  • Encrypt etcd at rest

Workloads:
  • Pod Security Standards (Restricted baseline)
  • Non-root containers
  • Read-only filesystem
  • No privileged containers
  • Resource limits (prevent resource abuse)
  • Network Policies (default deny, allow explicitly)

Supply Chain:
  • Signed images (Cosign)
  • Private registry only (no Docker Hub in prod)
  • Image scanning (Trivy in CI + admission webhook)
  • SBOM attached to images

Runtime:
  • Falco (anomaly detection)
  • No exec into production pods
  • Audit all API calls
  • Secrets encrypted (SOPS + KMS, or External Secrets)
```

**Q: What's the difference between SAST, DAST, SCA, and IAST?**
A:
| Type | When | What it does | Tools |
|------|------|-------------|-------|
| **SAST** | Build time | Analyzes source code for vulnerabilities | SonarQube, Semgrep, CodeQL |
| **SCA** | Build time | Checks dependencies for known CVEs | Snyk, Dependabot, Trivy |
| **DAST** | Runtime | Tests running app for vulnerabilities | OWASP ZAP, Burp Suite |
| **IAST** | Runtime | Agent inside app, monitors real requests | Contrast, Hdiv |

Use all four — they find different classes of bugs.

**Q: How do you implement secrets rotation with zero downtime?**
A:
1. **Dual-read**: App accepts both old and new secret simultaneously
2. **Update secret**: Rotate in Secrets Manager
3. **Propagate**: External Secrets Operator syncs new value to K8s
4. **Pods reload**: Either restart or watch for secret changes
5. **Verify**: Confirm app is using new secret
6. **Revoke old**: Delete previous secret version

For databases (RDS + Secrets Manager):
- Secrets Manager has built-in rotation Lambda
- Creates new user, updates secret, drops old user
- RDS Proxy handles connection draining

---

## Compliance & Governance

**Q: How do you ensure compliance at scale (SOC2, PCI-DSS)?**
A:
- **Policy as code** — OPA/Rego, Sentinel, AWS Config Rules
- **Continuous compliance** — don't check quarterly, check continuously
- **Evidence collection** — automated (CloudTrail → S3 → audit dashboard)
- **Guardrails, not gates** — prevent non-compliance rather than detecting it after
- **SCPs** — prevent disabling CloudTrail, creating public S3 buckets
- **Tagging enforcement** — can't create resources without required tags
- **Drift detection** — alert when production doesn't match code

**Q: How do you handle a security incident in a containerized environment?**
A:
1. **Identify**: Falco alert, GuardDuty finding, or manual report
2. **Contain**: Network Policy to isolate pod, scale down to 0 replicas
3. **Preserve**: Don't delete — snapshot the pod's filesystem, keep logs
4. **Investigate**: Check image layers, audit API calls, trace network connections
5. **Eradicate**: Rebuild from known-good image, rotate all secrets the pod accessed
6. **Recover**: Redeploy clean version
7. **Post-mortem**: How did it get in? What was the blast radius?

---

*These questions separate senior engineers from mid-level. Practice explaining your THOUGHT PROCESS, not just the answer! 🎯*
