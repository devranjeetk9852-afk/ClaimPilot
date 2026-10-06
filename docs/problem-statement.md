# ClaimPilot — Problem Statement & User Stories

## Problem Statement

Insurance claims processing is a high-volume, document-heavy workflow that traditionally requires significant manual effort. Adjusters spend hours reading claim documents, cross-referencing policies, assessing fraud risk, and writing decision letters—creating bottlenecks that delay payouts and frustrate customers.

**The opportunity:** Use AI agents to automate the intake, triage and assessment phases, freeing adjusters to focus on complex edge cases and customer relationships. A multi-agent system can read documents, check policy coverage, score fraud risk and draft recommendations in seconds, with a human adjuster retaining full control over the decision.

**Why it matters:** 
- Customers get faster claim decisions (days instead of weeks).
- Adjusters process 3–5× more claims per day.
- The system learns from adjuster feedback, improving over time.
- Risk is contained: agents recommend, humans approve.

## User Stories

### Story 1: Customer Files a Claim
**As a** customer with an insurance claim  
**I want to** upload photos and documents describing my loss  
**So that** I can file my claim without visiting an office or mailing physical documents

**Acceptance criteria:**
- I can submit a claim with my policy number and a description.
- I can upload multiple files (photos, receipts, police reports, etc.) in one flow.
- The system acknowledges receipt and gives me a reference number.
- An automated intake agent immediately checks if my claim is complete, flagging missing documents.

**Out of scope:** paying the claim, issuing decision letters.

---

### Story 2: Adjuster Reviews Incoming Claims
**As an** insurance adjuster  
**I want to** see a prioritized queue of claims with agent recommendations  
**So that** I can focus on claims that need human judgment rather than routine approvals

**Acceptance criteria:**
- I see a dashboard listing all claims with status, date filed, and agent assessment.
- I can sort/filter by claim type, status, or risk score.
- Clicking on a claim shows:
  - Original documents and metadata
  - Agent reasoning (what the intake, coverage and fraud agents found)
  - A recommended decision (approve/deny/request more info)
  - Citations to policy rules and documents
- I can drill into agent traces and see exactly which documents and policy sections were consulted.

**Out of scope:** paying claims (backend process).

---

### Story 3: Agent Assesses Claim Coverage
**As** the Coverage Agent (AI)  
**I want to** read the claim description and documents, then check the customer's policy  
**So that** I can determine if the loss is covered under the policy

**Acceptance criteria:**
- I receive a claim with documents via a message.
- I call a tool to fetch the customer's policy and coverage limits.
- I call a tool to search the policy for relevant clauses (e.g., "theft," "damage," "deductible").
- I produce a structured assessment: "Is this loss covered? If yes, what is the deductible and coverage limit?"
- If a clause is ambiguous, I flag it for human review.

**Out of scope:** making the final decision; denying fraud-related claims (the Fraud Agent's job).

---

### Story 4: Agent Detects Fraud Risk
**As** the Fraud Agent (AI)  
**I want to** analyze claim history, policy patterns and claim details for red flags  
**So that** I can alert the adjuster to potential fraud before they approve a claim

**Acceptance criteria:**
- I receive a claim with the customer's full history.
- I check for suspicious patterns: sudden recent policy increase, multiple claims in short time, high claim value vs. deductible, inconsistent damage photos, etc.
- I produce a risk score (low/medium/high) with a brief explanation.
- For high-risk claims, I recommend manual review.

**Out of scope:** making the fraud determination (adjuster does); blocking claims automatically.

---

### Story 5: Adjuster Approves Claim and Receives Decision Letter
**As an** adjuster  
**I want to** approve or deny a claim in the dashboard, and the customer automatically receives a clear decision letter  
**So that** the customer knows the outcome without me having to draft an email

**Acceptance criteria:**
- I click "Approve" or "Deny" on a claim in the dashboard.
- The system immediately generates a customer-friendly decision letter (not legalese).
- The letter includes: decision, reason, coverage limits (if approved), next steps, and a contact number.
- The customer receives the letter by email.
- A record is written to the claim history and stored for audit.

**Out of scope:** appeals process; payment processing.

---

## Key Design Principles

1. **Agents recommend, humans approve.** No claim is approved without an adjuster's action.
2. **Transparency and citations.** When an agent refers to a policy section, the adjuster can click and see exactly which document and page.
3. **Graceful degradation.** If an agent fails or is unsure, the claim queues for manual review; the customer is never left in limbo.
4. **Learn from feedback.** Over time, adjuster decisions feed back into agent training and guardrails.

---

## Success Metrics (Post-MVP)

- **Time to decision:** average < 4 hours (vs. 5+ days today).
- **Agent accuracy:** coverage recommendations agree with adjuster decisions 95%+ of the time.
- **Throughput:** adjuster processes 50+ claims per day (vs. 10–15 today).
- **Cost:** < $0.50 in LLM costs per claim processed.

---

## Next Steps

- Week 1, Day 3–4: Translate these stories into a C4 Context and Container diagram.
- Week 1, Day 5: Create GitHub issues (one per story) and start the project board.