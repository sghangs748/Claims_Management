# ESCALATION & HUMAN ADJUSTER WORKBENCH

This stage handles all claims that cannot be reliably auto-settled. It ensures accuracy, compliance, auditability, and human oversight.

### 1. Escalation happens for the following conditions:

#### LLM vs Rule Engine Conflict

Example:

Rules engine says claim is covered
LLM says clause interpretation = not covered
This is a high-risk area → human intervention needed.

#### Policy Clause Ambiguity

Some policies contain:

vague definitions
exceptions within exceptions
conditional statements
LLM may return low confidence → send to human.

#### Severe Damage (> threshold)

Structural damage beyond ₹50,000 or ₹1 lakh often requires:
multiple workshop quotes
inspection reports
higher scrutiny
Fraud Score Borderline (0.5–0.8)
Not clearly fraudulent, but suspicious.

#### Unclear or Conflicting Evidence

Examples:

blurry photos
night-time images
mismatch between damage and incident narrative

#### Missing Data Cases

garage price not available
rural areas with low digital footprint

### 2. WORKBENCH QUEUE SYSTEM

This manages how escalated claims are routed to the correct human adjusters.

#### Queue System to ensure:

Load balancing across adjusters
Region-based routing
SLA-prioritized case assignment
Specialist routing (motor vs property vs medical)
Audit tracking for every claim assignment

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

FNOL
policy coverage
damage assessment
fraud findings
Example output:

“Claim involves right-side impact with bumper + headlamp damage.
Weather shows heavy rainfall at time of accident.
Fraud score borderline due to similar previous claim.”

#### Evidence Analysis Agent

Spot anomalies:

inconsistent timestamps
mismatched geolocation
duplicate images used in past claims
EXIF metadata showing image tampering
lens distortion patterns

LLM highlights:

“Image #2’s timestamp predates accident by 3 days.”

#### Policy Reasoning Agent

Adjuster can ask:

“Is cracked windshield covered under my policy?”

LLM (backed by RAG) responds with exact clause reference:

“Windshield damage is covered under Add-On Code W2: Glass Shield.”

#### Recommendation Agent

Based on combined signals:

damage cost
fraud risk
policy rules
LLM reasoning

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

adjuster ID
timestamp
decision type
LLM assistance logs
change from auto-estimate (delta)
justification text
policy references used
fraud signals considered

### 7. FINAL EVENT EMISSION

HumanSettlementCompleted {
  claim_id,
  approved_amount,
  adjuster_id,
  notes,
  timestamp
}