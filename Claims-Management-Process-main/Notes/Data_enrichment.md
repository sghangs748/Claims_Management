# Data Enrichment

Data Enrichment takes the initial claim snapshot and enriches it with authoritative, contextual, and derived data (policy details, customer 360, asset metadata, geolocation, weather, third-party risk signals and similarity hits) so downstream ML, rules and human workflows have the facts they need.

Trigger by : ClaimIntakeCompleted event (published to the Event Bus after intake finishes and files are uploaded).

Outcome: DataEnrichmentCompleted event with an enrichment snapshot saved to DB and a pointer for downstream consumers (Triage, Fraud, Damage, etc).

### 1. Fetch claim snapshot:

Call OLTP Claim DB (read replica) to retrieve claim, uploaded file refs, minimal metadata

### 2. Parallelize enrichment tasks

### Policy Service:

Checks: 
1. coverage type 
2. limits 
3. deductibles 
4. exclusions 
5. product type 
6. add-ons (zero dep, engine protector, storm protection, etc.)

### Customer 360:

fetch:
1. past claims 
2. risk score 
3. KYC 
4. behavior profile 
5. renewal history 

### Asset Master data:

Depending on product type: 

Motor Insurance:
1. Make/model 
2. CC/kW 
3. Year 
4. IDV 
5. Parts price sheet 

Property Insurance:
1. Building type 
2. Construction year 
3. Fire risk category 
4. Previous survey reports 

### Historical Claim Similarity (Vector DB):

Embeddings from intake stage allow: 

1. find similar past claims 
2. anomaly detection 
3. fraud pattern matching 
4. fast decision support 

### Geo APIs:

Reverse-geocodes incident location → identifies: 

1. nearest city 
2. crime/fraud hotspot 
3. accident-prone region 
4. flood-zone classification 

### Weather APIs:

1. Cross-check: 
2. rain intensity 
3. storm alerts 
4. flood warnings 
5. temperature 
6. visibility 

### Third-party Risk APIs:

APIs for: 
1. fraud blacklists 
2. stolen vehicle DB 
3. identity scoring (e.g., Perfios, Bureau) 
4. accident hotspots 

### 3. Save All Metadata 

One consolidated enriched claim object: 

1. policy snapshot 
2. customer360 
3. geo + weather 
4. risk score 
5. asset metadata 
6. embedding similarity hits 

### 4. Event Published 

### DataEnrichmentCompleted triggers: 

1. triage 
2. fraud detection 
3. adjuster assignment 