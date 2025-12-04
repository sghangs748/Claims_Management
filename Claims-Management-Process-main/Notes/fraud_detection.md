# Fraud Detection 

### 1. Trigger: The fraud workflow starts when Triage sends: 

TriageCompleted {claim_id} 

### 2. Claim Snapshot Loaded:
Fraud service collects everything needed to score and explain:

1. Claim metadata: claim_id, policy_id, timestamps, claim_amount, submission channel, device fingerprint.
2. OCR text: from document ingestion (driver DL, RC, invoices, FIR).
3. Weather & Geo: enrichment snapshot (flood zone, time-of-day).
4. Policy & Customer360: policy status, past claims, risk tier, KYC, payment history.
5. Embeddings references: pointers to document/image/text embeddings in Vector DB.

### 3. ML Fraud Model — supervised approach:

Quick probabilistic signal from historical data to identify high-probability fraud.

### Model type:

Tabular models: LightGBM / XGBoost for initial production (fast, interpretable, robust).

### Features (categorize & examples)

#### A. Claim-level

1. claim_amount
2. time_since_policy_inception_days
3. time_of_day (hour)
4. incident_vs_report_delay (minutes/hours/days)
5. claim_channel (web/agent/phone)
6. number_of_photos_uploaded

#### B. Customer-level / behavior

1. past_claim_count_1y
2. average_claim_amount
3. policy_renewals_count
4. dob_age
5. kyc_status

#### C. Document & media signals

1. OCR text length
2. presence_of_specific_keywords (e.g., "flood", "hit-and-run")
3. num_unique_documents vs expected docs for claim_type

#### D. Device / submission

1. device_fingerprint_risk_score (from device risk service)
2. ip_geolocation_match_policy_address boolean

#### E. Temporal & velocity

1. claims_in_last_30_days_by_phone_number
2. claims_from_same_account_last_24h

#### F. Derived / Aggregated

1. similarity_top_score (from vector search)
2. graph_degree_of_policyholder_node

### Labels (training target)

label = 1 for confirmed fraud cases 
label = 0 for confirmed legitimate

### 4. Vector Similarity (Embedding Search)

Purpose: detect reuse or semantic similarity across multimodal artifacts.

Compute embeddings at ingestion; store in Vector DB (Pinecone/Qdrant/Weaviate/Chroma).

Fraud service queries top-k similar vectors (image or text) and retrieves:
similar_claim_id, sim_score, field (which object matched)

If sim_score > threshold (e.g., 0.85) for cross-claim image → strong duplicate signal.

### 5. Graph Network Analysis:

Purpose: find collusion rings, shared actors (repair shops, garage owners, agents), repeated bank accounts.

### Graph design (core nodes & edges)

#### Nodes:

policyholder_id
claim_id
phone_number
email
device_fingerprint
bank_account
repair_shop_id
agent_id
garage_id
document_hash (perceptual hash)

#### Edges:

policyholder -> submitted -> claim
claim -> used_document -> document_hash
claim -> paid_to -> bank_account
claim -> serviced_by -> repair_shop
policyholder -> has_phone -> phone_number
phone_number -> used_in -> claim (same phone triggers detection)

#### Queries / patterns

Hops analysis: find shortest path length between two claims via shared nodes.

Community detection: detect dense clusters (Louvain or Label Propagation).

Anomaly scoring: node centrality, betweenness; if a repair_shop is involved in many high-risk clusters → suspicious.

Temporal pattern: windowed graph for last 12 months.

#### Output

graph_risk_score (0–1)
suspicious_node_list (repair shops, bank accounts)
shared_entity_count (number of claims linked)

### 6. 3rd-Party Risk APIs:

Purpose: authoritative external checks.

#### Typical APIs

1. Stolen vehicle DB (government / Vahan-like)
2. Police FIR verification / FIR upload verification
3. National ID / PAN / Aadhaar verification (where legal)
4. Blacklists (known fraudsters)
5. Credit bureau / AML checks

### 7. LLM Reasoning — "Fraud analyst assistant":

Purpose: handle narrative inconsistencies and produce human-readable reasoning

ML/graph find statistical signals; LLM interprets nuance in text, timeline, and multi-modal contradictions.

#### Inputs to LLM

1. Narrative text (claim description)
2. OCR text snippets (e.g., invoice contents)
3. Vision labels and confidences
4. Policy snippet via RAG (relevant coverage)
5. Similarity & graph findings (bulleted)
6. Key metadata (timestamps, location)

#### Output

1. llm_verdict: textual reasoning
2. llm_score (confidence proxy)
3. suggested_questions for human agent

### 8. Aggregation of All Fraud Signals:

Goal: combine heterogeneous signals into one CompositeFraudScore

#### Signals to aggregate

1. fraud_score_ml (tabular model, 0–1)
2. graph_score (0–1)
3. external_score (0–1)
4. similarity_anomaly_score (0–1)
5. llm_score (0–1 or proxy)

### 9. Routing Logic (decision thresholds)

composite_score > 0.8 → Route to Fraud Investigation Desk (High priority; open full investigation case).

0.5 ≤ composite_score ≤ 0.8 → Manual Re-evaluation (Adjuster review or enhanced verification questions).

composite_score < 0.5 → Pass to next stage (damage assessment / settlement flow).

#### Also

If external_flag == "stolen_vehicle" → escalate immediately regardless of score.

If claim_amount > high_value_threshold AND composite_score > 0.3 → escalate to senior reviewer.

### 10. Final Event Emitted
Publish: FraudCheckCompleted { claim_id, composite_score, decision, signals, model_version, reasoning_ref }