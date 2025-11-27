# Initial Assessment & Triage

This stage is the first point in the workflow where the system tries to "understand" the claim and classify it.
Its output determines routing, priority, SLAs, and what specialist services will process the claim next.

Rapidly classify claim severity & complexity, route to auto-flow or human, and estimate resource needs.

### The Initial Assessment & Triage Service performs the following:

✔️ Claim Understanding

Interprets the structured fields (claim type, incident date, policy number, etc.)

Consumes the enriched unstructured data (summaries, extractions, embeddings from earlier stages)

✔️ Claim Classification

Is this a motor insurance claim?
A health claim?
A property damage claim?
A subrogation case?
Fraud-suspicious?

✔️ Routing / Assignment Decisions

Which downstream pipeline should handle it?
Should a human reviewer be involved early?
What SLA rules apply?

✔️ Risk & Priority Scoring

Uses ML models (risk scoring, fraud heuristics).
Determines whether this should be fast-tracked or escalated.

✔️ Creates a Standardized Internal Object

This service produces the canonical:
ClaimAssessmentBundle
containing:

claim category (motor/health/home/etc)
risk score (0–1)
priority score (0–100)
recommended workflow route (fast-track / manual review / expert)
flags (fraud suspicion / missing docs / policy mismatch)
assessment_explanation (LLM explainability)

This bundle is published back to the event bus.

### Flow
1. Load Policy + Claim Data
2. Rule-Based Pre-Checks

### Before ML:

Policy active?
Coverage valid for claim type?
Fraud blacklist?
Document completeness?
Duplicate claim check (via embeddings similarity search)

### 3. ML Classification Models

A. Claim Type Classifier Model

Output: motor / health / property / travel / etc

B. Risk Scoring Model

Gradient Boosted Trees or NN → outputs risk probability

C. Fraud Heuristic Layer

### 4. LLM Assessment Explanation
### 5. Workflow Routing Logic:

Example:
IF fraud_score > 0.7:
    route = "fraud_investigation"

ELSE IF risk_score < 0.2 AND claim_amount < 50k:
    route = "fast_track"

ELSE:
    route = "manual_adjuster_review"

### 6. Output Event Published