# Day 55 — Amazon Leadership Principles (STAR)

**Week 8 · Saturday · ~2–3 hours**  
Prepare 8–12 stories mapping to Amazon’s 16 Leadership Principles (LPs).

---

# STAR Format Template

| Section | Length | Content |
|---------|--------|---------|
| **S** — Situation | 2–3 sentences | Team, company, context (no jargon dump) |
| **T** — Task | 1–2 sentences | Your responsibility, stakes |
| **A** — Action | 60% of answer | **What YOU did** (I, not we) — decisions, tradeoffs |
| **R** — Result | 2–3 sentences | Metrics, business impact, what you learned |

**Timing:** 2–3 min spoken per story · **Deep dive:** interviewer may spend 10 min on one LP.

---

# Story Bank Worksheet

Fill one row per story (aim for 10+).

| # | LP | Story title (3 words) | Situation (1 line) | Key metric / result |
|---|-----|----------------------|--------------------|---------------------|
| 1 | Customer Obsession | | | |
| 2 | Ownership | | | |
| 3 | Invent and Simplify | | | |
| 4 | Are Right, A Lot | | | |
| 5 | Learn and Be Curious | | | |
| 6 | Hire and Develop the Best | | | |
| 7 | Insist on Highest Standards | | | |
| 8 | Think Big | | | |
| 9 | Bias for Action | | | |
| 10 | Frugality | | | |
| 11 | Earn Trust | | | |
| 12 | Dive Deep | | | |
| 13 | Have Backbone; Disagree and Commit | | | |
| 14 | Deliver Results | | | |
| 15 | Strive to be Earth’s Best Employer | | | |
| 16 | Success and Scale Bring Broad Responsibility | | | |

---

# Leadership Principles — Quick Map

| LP | Look for |
|----|----------|
| Customer Obsession | Started from customer pain, worked backward |
| Ownership | Long-term fix, no “not my job” |
| Invent and Simplify | Removed complexity, novel solution |
| Are Right, A Lot | Judgment with incomplete data |
| Learn and Be Curious | Self-taught skill, applied fast |
| Hire and Develop | Mentored, raised bar on team |
| Insist on Highest Standards | Refused to ship mediocrity |
| Think Big | 10x vision, ambitious plan |
| Bias for Action | Calculated risk, shipped fast |
| Frugality | More with less |
| Earn Trust | Transparent, admitted mistake |
| Dive Deep | Root cause, data/logs |
| Backbone | Disagreed with manager, then committed |
| Deliver Results | Hit deadline despite obstacles |

---

# Example Story 1 — Customer Obsession

**Prompt:** Tell me about a time you went above and beyond for a customer.

**S:** At [Company], our B2B dashboard had 40% drop-off after login because load time exceeded 30 seconds for large accounts.

**T:** As backend lead, I owned API latency for the home dashboard used by 2,000 enterprise users.

**A:**

- Interviewed 5 churned accounts; traced pain to N+1 queries and unbounded aggregations  
- Prioritized fixes by revenue impact; paired with PM on “good enough” MVP scope  
- Added DB indexes, rewrote ORM fetch joins, introduced Redis cache for aggregate stats  
- Shipped behind feature flag; monitored p99 with CloudWatch alarms  

**R:** p99 dropped from 28s to 1.2s; support tickets for “slow dashboard” fell 70% in one quarter; renewal rate for that segment improved 8%. Learned to validate with customer calls before optimizing the wrong layer.

---

# Example Story 2 — Ownership

**Prompt:** Tell me about something broken that you fixed without being asked.

**S:** Nightly batch jobs failed silently for 3 weeks; finance reports were wrong but no alert fired.

**T:** I noticed discrepancies during on-call handoff — not officially owner of the batch system.

**A:**

- Took ownership despite another team owning the scheduler  
- Added idempotent job steps + dead-letter queue  
- Built dashboard for last success timestamp per job  
- Documented runbook and trained adjacent team  

**R:** Zero silent failures in 6 months; finance regained trust in reports. Team adopted my monitoring pattern for other batches.

---

# Example Story 3 — Dive Deep

**Prompt:** Tell me about a time you solved a problem by digging into details.

**S:** Payment API showed intermittent 500 errors (~0.3%) only in production EU region.

**T:** I was on-call; SRE escalated after payment partner threatened SLA breach.

**A:**

- Correlated logs by `trace_id`; found `DuplicateKeyException` on idempotency table  
- Traced to race: two pods retrying same webhook without distributed lock  
- Implemented DB unique constraint + `ON CONFLICT` upsert and short Redis lock  
- Replayed failed webhooks from DLQ with script  

**R:** Error rate → 0%; recovered €120K at-risk transactions. Now standard pattern for all webhook handlers.

---

# Example Story 4 — Bias for Action

**Prompt:** Tell me about a time you made a decision with incomplete information.

**S:** Launch deadline in 2 weeks; third-party KYC API integration was undefined.

**T:** I could wait for legal contract or ship MVP for internal beta.

**A:**

- Proposed stub adapter + feature flag; parallel track for real API  
- Defined contract tests both sides must pass  
- Daily 15-min sync with vendor; documented fallback manual review for 50 beta users  

**R:** Beta launched on time; real API swapped in 10 days later with zero downtime. Leadership used approach for two other vendor integrations.

---

# Example Story 5 — Have Backbone; Disagree and Commit

**Prompt:** Tell me about disagreeing with your manager or team.

**S:** Team planned microservices split for monolith before traffic justified complexity.

**T:** I was senior engineer on architecture review.

**A:**

- Wrote 2-page doc: cost of ops, deployment, debugging vs expected scale (10x in 12 mo)  
- Presented data: current 200 RPS, p99 stable in modular monolith  
- Disagreed respectfully in staff meeting; proposed bounded contexts inside monolith first  
- After VP chose microservices anyway, I committed: owned extraction of User service  

**R:** User service extracted cleanly; I was go-to for cross-team contracts. Learned to disagree with data, then execute fully.

---

# “Tell Me About Yourself” (2 min)

Structure:

1. **Present** — role, years, domain (30 sec)  
2. **Past** — 2 relevant achievements (60 sec)  
3. **Future** — why this role/company (30 sec)  

Avoid: childhood, unrelated hobbies, reading resume verbatim.

---

# “Why Amazon?” — Framework

- Scale impact (millions of customers)  
- Engineering bar (LPs, ownership culture)  
- Specific team/product if known  
- Personal tie: past AWS use, customer obsession story  

---

# Weakness Question

Pick real weakness + **active mitigation**:

> “I sometimes dive too deep before aligning stakeholders. Now I timebox investigation to 2 hours and share a short status doc before continuing.”

---

# Questions to Ask Interviewer

1. What does success look like in the first 6 months for this role?  
2. How does the team balance operational load vs feature work?  
3. What is the biggest technical challenge the team faces today?  
4. How are LPs reflected in performance reviews on this team?  

---

# Day 55 Practice Plan

| Block | Activity |
|-------|----------|
| 60 min | Fill story bank worksheet (10 stories) |
| 60 min | Write full STAR for top 5 LPs (Customer, Ownership, Dive Deep, Deliver Results, Bias for Action) |
| 30 min | Record yourself — 2 stories, check for “we” vs “I” |
| 30 min | Peer mock: random LP, 3 min answer |

---

# Red Flags to Avoid

- No metrics in Result  
- Team did everything (“we”) — interviewer can't assess **you**  
- Blaming others without ownership  
- Illegal / unethical examples  
- Stories where you had no agency (observer only)  

---

# LP → Likely Technical Overlap (Backend)

| LP | Tie to engineering |
|----|-------------------|
| Dive Deep | Debugging prod, profiling |
| Deliver Results | Shipped feature under deadline |
| Insist on Highest Standards | Code review, testing bar |
| Invent and Simplify | Refactored over-engineered design |
| Ownership | On-call, tech debt without ticket |

---

*End of Day 55 — Behavioural*
