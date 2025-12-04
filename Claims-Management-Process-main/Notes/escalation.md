# ESCALATION & HUMAN ADJUSTER WORKBENCH

This stage handles all claims that cannot be reliably auto-settled. It ensures accuracy, compliance, auditability, and human oversight.

### 1. Escalation happens for the following conditions:

#### LLM vs Rule Engine Conflict

Example:

1. Rules engine says claim is covered
2. LLM says clause interpretation = not covered
3. This is a high-risk area → human intervention needed.

#### Policy Clause Ambiguity

Some policies contain:

1. vague definitions
2. exceptions within exceptions
3. conditional statements
4. LLM may return low confidence → send to human.

#### Severe Damage (> threshold)

1. Structural damage beyond ₹50,000 or ₹1 lakh often requires:
2. multiple workshop quotes
3. inspection reports
4. higher scrutiny
5. Fraud Score Borderline (0.5–0.8)
6. Not clearly fraudulent, but suspicious.

#### Unclear or Conflicting Evidence

Examples:

1. blurry photos
2. night-time images
3. mismatch between damage and incident narrative

#### Missing Data Cases

1. garage price not available
2. rural areas with low digital footprint

### 2. WORKBENCH QUEUE SYSTEM

This manages how escalated claims are routed to the correct human adjusters.

#### Queue System to ensure:

1. Load balancing across adjusters
2. Region-based routing
3. SLA-prioritized case assignment
4. Specialist routing (motor vs property vs medical)
5. Audit tracking for every claim assignment

### 3. ADJUSTER UI WORKFLOW

When an adjuster clicks on a claim in UI, the system fetches all relevant information.

UI Fetches:
A. Claim Data

FNOL
timestamp
location & geo-coordinates

B. Policy Data

coverage
add-ons
exclusions
deductibles

C. Assessment Data

damage classification
vision AI masks
part-level segmentation
severity score

D. Fraud Indicators

ML fraud score
graph anomalies
vector similarity anomalies
LLM-computed inconsistencies

E. Evidence

images, videos
documents (OCR extracted text)
CCTV or workshop uploads

F. Weather & Geo

rainfall intensity
road condition index
historical traffic patterns

G. LLM Summary (Auto-generated)

This helps reduce adjuster effort dramatically.

### 4. LLM Assistance Inside the Workbench

Agentic AI plays a major role here.

#### Claim Summary Agent

Summarizes:

1. FNOL
2. policy coverage
3. damage assessment
4. fraud findings

Example output:

“Claim involves right-side impact with bumper + headlamp damage.
Weather shows heavy rainfall at time of accident.
Fraud score borderline due to similar previous claim.”

#### Evidence Analysis Agent

Spot anomalies:

1. inconsistent timestamps
2. mismatched geolocation
3. duplicate images used in past claims
4. EXIF metadata showing image tampering
5. lens distortion patterns

LLM highlights:

“Image #2’s timestamp predates accident by 3 days.”

#### Policy Reasoning Agent

Adjuster can ask:

“Is cracked windshield covered under my policy?”

LLM (backed by RAG) responds with exact clause reference:

“Windshield damage is covered under Add-On Code W2: Glass Shield.”

#### Recommendation Agent

Based on combined signals:

1. damage cost
2. fraud risk
3. policy rules
4. LLM reasoning

#### LLM suggests next steps:

“Recommend partial approval because right headlamp appears aftermarket.
Defer to garage quote for bumper refill.”

#### Auto-Drafted Decision Notes

LLM drafts full settlement note:

“Based on inspection and supporting evidence, approve ₹18,200 settlement.
Deductible applied: ₹1,000. Depreciation applied: 15% for plastic bumper.”

Adjuster only edits & approves.

### 5. ADJUSTER DECISION WORKFLOW

The human makes one of the following decisions:

#### Approve Full Settlement

Typical cases:

Clear damage
Strong evidence
Low fraud risk

#### Approve Partial

Reasons:

aftermarket parts used
depreciation applied
partial coverage

#### Request More Info

Options:

ask for more photos
request police report
request repair estimate

#### Schedule Physical Inspection

Triggered when:

structural damage
high-value claims
unclear images

#### Reject Claim

With clear justification:

non-covered peril
policy expired
fraud suspected

### 6. APPROVAL SERVICE (Audit Layer)

Every decision is stored with full auditability.

Stored Data

1. adjuster ID
2. timestamp
3. decision type
4. LLM assistance logs
5. change from auto-estimate (delta)
6. justification text
7. policy references used
8. fraud signals considered

### 7. FINAL EVENT EMISSION

HumanSettlementCompleted {
  claim_id,
  approved_amount,
  adjuster_id,
  notes,
  timestamp
}