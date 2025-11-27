# Data Enrichment

Data Enrichment takes the initial claim snapshot and enriches it with authoritative, contextual, and derived data (policy details, customer 360, asset metadata, geolocation, weather, third-party risk signals and similarity hits) so downstream ML, rules and human workflows have the facts they need.

Trigger by : ClaimIntakeCompleted event (published to the Event Bus after intake finishes and files are uploaded).

Outcome: DataEnrichmentCompleted event with an enrichment snapshot saved to DB and a pointer for downstream consumers (Triage, Fraud, Damage, etc).

1. Fetch claim snapshot:

Call OLTP Claim DB (read replica) to retrieve claim, uploaded file refs, minimal metadata

2. Parallelize enrichment tasks

Policy Service:

Checks: 
coverage type 
limits 
deductibles 
exclusions 
product type 
add-ons (zero dep, engine protector, storm protection, etc.)

Customer 360:

fetch:
past claims 
risk score 
KYC 
behavior profile 
renewal history 

Asset Master data:

Depending on product type: 

Motor Insurance:
    Make/model 
    CC/kW 
    Year 
    IDV 
    Parts price sheet 

Property Insurance:
    Building type 
    Construction year 
    Fire risk category 
    Previous survey reports 

Historical Claim Similarity (Vector DB):

Embeddings from intake stage allow: 

find similar past claims 
anomaly detection 
fraud pattern matching 
fast decision support 

Geo APIs:
Reverse-geocodes incident location → identifies: 

nearest city 
crime/fraud hotspot 
accident-prone region 
flood-zone classification 

Weather APIs:

Cross-check: 
rain intensity 
storm alerts 
flood warnings 
temperature 
visibility 

Third-party Risk APIs:

APIs for: 
fraud blacklists 
stolen vehicle DB 
identity scoring (e.g., Perfios, Bureau) 
accident hotspots 

3. Save All Metadata 

One consolidated enriched claim object: 

policy snapshot 
customer360 
geo + weather 
risk score 
asset metadata 
embedding similarity hits 

4. Event Published 

DataEnrichmentCompleted triggers: 

triage 
fraud detection 
adjuster assignment 