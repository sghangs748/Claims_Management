# Auto Settlement Engine 

This stage begins after Damage Assessment finishes and the claim has:

1. Itemized repair cost
2. Parts list
3. Severity + Confidence
4. Fraud score
5. Damage Assessment Completed event

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

1. what is covered
2. what is NOT covered
3. how much depreciation & deductible applies

#### Inputs

1. Policy number
2. Claim cause (accident, theft, fire, flood)
3. Vehicle age

#### Loaded from database/cache

1. Coverage type (Comprehensive / Third-party / Collision)
2. Add-ons
3. Zero Depreciation (Zero-Dep)
4. NCB Protector
5. Engine Protector
6. Consumables Cover
7. Return-to-Invoice
8. Deductibles
9. Compulsory deductible
10. Voluntary deductible
11. Sum insured
12. Exclusions
13. Drunk driving
14. Mechanical failure
15. Wear/tear
16. Non-accidental loss
17. IDV (Insurance Declared Value)
18. Policy validity dates

### 3. DETERMINISTIC RULES ENGINE

Not everything needs AI. Many decisions are 100% deterministic and must be auditable

#### Immediate rejection

1. Policy expired on date of accident
2. Non-covered peril. E.g., Claim cause = flood, but no Flood/RAD cover
3. Driver not licensed

#### small claims

If total estimate < compulsory deductible → no payout

#### Zero Depreciation add-on

If Zero Dep is active:

1. Depreciation = 0% for plastic, metal, rubber, fiber parts
2. Except tires and batteries (depends on insurer)

#### NCB Protector

If NCB protector active:

Do NOT reduce No Claim Bonus, even after payout

### 4. LLM INTERPRETATION

Policies have complex clauses that are:

1. legally heavy
2. ambiguous
3. conditional

LLM Responsibilities

1. Interpret clause text
2. Retrieve relevant policy paragraph using embeddings
3. Check if claim is covered
4. Explain the decision (for audit trail)
5. Compare claim incident with policy conditions

### 5. ESTIMATE RECONCILIATION

After policy interpretation, the system reconciles:

1. Parts
2. Labour
3. Paint
4. Taxes
5. OEM adjustments
6. Depreciation (if allowed)
7. Deductibles
8. Add-on overrides

Why reconciliation is needed

1. Damage Assessment → gives total cost
2. Policy Engine → gives coverage logic
3. Auto Settlement → must combine both

#### This step ensures:

The cost components align with policy coverage rules

### 6. COMPUTE FINAL SETTLEMENT AMOUNT

1. claimable = eligible_cost
2. claimable -= compulsory_deductible
3. claimable -= voluntary_deductible
4. claimable -= depreciation (if zero-dep not active)
5. claimable = min(claimable, policy_limit)

1. Zero Dep removes depreciation
2. Glass-only claims may waive deductible
3. Engine Protector removes exclusion for hydrolock
4. Consumables add-on covers nuts, bolts, fluids
5. Return-to-Invoice includes on-road price, not IDV

### 7. AUTO DECISIONING

The final decision determines how the workflow proceeds.

#### Auto Approve When:

1. Fraud score low
2. Vision + Damage Assessment confidence high
3. Rule Engine → coverage = yes
4. LLM → no conflict
5. Settlement < threshold. E.g., ₹25,000 (configurable)
6. Small/simple claims get instant payout.

#### Escalate to Human Assessor When:

1. Rule engine and LLM disagree
2. Fraud score borderline
3. Damage Assessment confidence low
4. High-value claims > ₹1 lakh
5. Engine damage (usually complex)
6. Multi-part collision
7. Potential total-loss