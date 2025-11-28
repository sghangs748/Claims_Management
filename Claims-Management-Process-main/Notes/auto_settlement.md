# Auto Settlement Engine 

This stage begins after Damage Assessment finishes and the claim has:

Itemized repair cost
Parts list
Severity + Confidence
Fraud score
Damage Assessment Completed event

This stage determines how much to pay, whether to pay automatically, or route for human review.

### 1. Trigger – From Damage Assessment

DamageAssessmentCompleted { total_estimate, confidence, fraud_score }

We only run auto-settlement after we know damage cost.

If fraud_score < threshold AND damage_assessment_confidence > threshold:
      → Start Auto Settlement
Else:
      → Manual/Expert review


### 2. POLICY FETCH SERVICE

After receiving the assessment, the service pulls all relevant policy data. Settlement requires precise knowledge of:

what is covered
what is NOT covered
how much depreciation & deductible applies

#### Inputs

Policy number
Claim cause (accident, theft, fire, flood)
Vehicle age

#### Loaded from database/cache

Coverage type (Comprehensive / Third-party / Collision)
Add-ons
Zero Depreciation (Zero-Dep)
NCB Protector
Engine Protector
Consumables Cover
Return-to-Invoice
Deductibles
Compulsory deductible
Voluntary deductible
Sum insured
Exclusions
Drunk driving
Mechanical failure
Wear/tear
Non-accidental loss
IDV (Insurance Declared Value)
Policy validity dates

### 3. DETERMINISTIC RULES ENGINE

Not everything needs AI. Many decisions are 100% deterministic and must be auditable

#### Immediate rejection

Policy expired on date of accident
Non-covered peril
e.g., Claim cause = flood, but no Flood/RAD cover
Driver not licensed

#### mall claims

If total estimate < compulsory deductible → no payout

#### Zero Depreciation add-on

If Zero Dep is active:

Depreciation = 0% for plastic, metal, rubber, fiber parts
Except tires and batteries (depends on insurer)

#### NCB Protector

If NCB protector active:

Do NOT reduce No Claim Bonus, even after payout

### 4. LLM INTERPRETATION

Policies have complex clauses that are:

legally heavy
ambiguous
conditional

LLM Responsibilities

Interpret clause text
Retrieve relevant policy paragraph using embeddings
Check if claim is covered
Explain the decision (for audit trail)
Compare claim incident with policy conditions

### 5. ESTIMATE RECONCILIATION

After policy interpretation, the system reconciles:

Parts
Labour
Paint
Taxes
OEM adjustments
Depreciation (if allowed)
Deductibles
Add-on overrides

Why reconciliation is needed

Damage Assessment → gives total cost
Policy Engine → gives coverage logic
Auto Settlement → must combine both

#### This step ensures:

The cost components align with policy coverage rules

### 6. COMPUTE FINAL SETTLEMENT AMOUNT

claimable = eligible_cost
claimable -= compulsory_deductible
claimable -= voluntary_deductible
claimable -= depreciation (if zero-dep not active)
claimable = min(claimable, policy_limit)

Zero Dep removes depreciation
Glass-only claims may waive deductible
Engine Protector removes exclusion for hydrolock
Consumables add-on covers nuts, bolts, fluids
Return-to-Invoice includes on-road price, not IDV

### 7. AUTO DECISIONING

The final decision determines how the workflow proceeds.

#### Auto Approve When:

Fraud score low
Vision + Damage Assessment confidence high
Rule Engine → coverage = yes
LLM → no conflict
Settlement < threshold
e.g., ₹25,000 (configurable)
Small/simple claims get instant payout.

#### Escalate to Human Assessor When:

Rule engine and LLM disagree
Fraud score borderline
Damage Assessment confidence low
High-value claims > ₹1 lakh
Engine damage (usually complex)
Multi-part collision
Potential total-loss