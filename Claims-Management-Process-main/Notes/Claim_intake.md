# Claim Intake (Customer / Agent Interface)

1. User triggers intake via mobile/web/chat/IVR/telemetry. 

2. Gateway authenticates; returns pre-signed upload URLs for media. 

3. Intake layer validates minimal required fields and guides user (multi-angle photo prompts, hints). 

4. Intake agent (LLM-based conversational assistant) optionally collects missing data and calls backend endpoints. 

LLM gives frictionless guided intake, increasing completeness & quality of evidence (improves downstream automation rate). 

LLM: intake assistant — natural conversation, dynamic question flow, intent parsing (fast response; can be run with smaller LLMs for cheaper ops). 

Agentic AI: intake agent acts as an agent that decides: “ask for DL → if not provided, request photo → pre-populate policy lookup” (function calling + orchestration).

Agentic behavior reduces need for human orchestration and can pre-flight the claim before it touches backend services. 

## Flow

### 1. Customer/Agent Opens the Claim Web Portal

The frontend UI collects:

1. Policy Number
2. Accident Date/Time
3. Accident Location
4. Basic Description
5. Optional Photos (vehicle front, back, RC, driving license)
6. Contact Information
7. Who was driving

### 2. API Gateway Routes Request 

Instead of sending directly to FastAPI, the browser sends post request.This call hits the API Gateway, not the backend services 

API Gateway responsibilities here:

1. Verify customer JWT token 
2. Validate headers
3. Apply rate limiting
4. Block malicious payloads (WAF)
5. Route request to claim-intake-service

### 3. API Gateway → Claim Intake Service (FastAPI):

Gateway forwards request internally: The Claim Intake Service now begins its job.

Claim Intake Service responsibilities:

1. Validate form fields
2. Verify policy number format
3. Basic eligibility (policy active? customer valid?)
4. Use LLM for Smart Intake
5. Assign unique claim_id
6. Save initial FNOL (First Notice Of Loss) to database
7. Generate S3 Pre-Signed URLs for uploading photos.

LLM Performs Smart Intake 
LLM assists by: 

1. extracting structured data 
2. normalizing addresses, dates, narratives 
3. identifying missing details 
4. generating follow-up questions 

This dramatically reduces manual work. 

Now the portal knows the claim is created, but documents still need to be uploaded.

### 4. Customer Uploads Photos → S3 Direct Upload

User uploads:

1. Damage photo front
2. Damage photo rear
3. RC book
4. Driving license

These go directly to S3, not through the backend.
 

### 5. Initial Claim Record Created 

Stored in OLTP database. 

### 6. Claim Intake Service → Publishes "Claim.Created" Event 

ClaimIntakeCompleted event triggers: 

1. document ingestion 
2. enrichment 
3. triage 
4. fraud baseline scoring 

### 7. Customer Gets Confirmation 

Claim ID returned, ready for next steps. 

### Tech Used:
1. API Gateway: AWS API Gateway/Kong
2. Frontend: React(Web),Twilio/WhatsApp API for chat
3. LLM: GPT-5/Claude
4. Storage: S3 for media,Postgres for claim metadata.