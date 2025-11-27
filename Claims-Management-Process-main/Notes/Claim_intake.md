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

Policy Number
Accident Date/Time
Accident Location
Basic Description
Optional Photos (vehicle front, back, RC, driving license)
Contact Information
Who was driving

### 2. API Gateway Routes Request 

Instead of sending directly to FastAPI, the browser sends post request.This call hits the API Gateway, not the backend services 

API Gateway responsibilities here:

Verify customer JWT token 
Validate headers
Apply rate limiting
Block malicious payloads (WAF)
Route request to claim-intake-service

### 3. API Gateway → Claim Intake Service (FastAPI):

Gateway forwards request internally: The Claim Intake Service now begins its job.

Claim Intake Service responsibilities:

Validate form fields
Verify policy number format
Basic eligibility (policy active? customer valid?)
Use LLM for Smart Intake
Assign unique claim_id
Save initial FNOL (First Notice Of Loss) to database
Generate S3 Pre-Signed URLs for uploading photos.

LLM Performs Smart Intake 
LLM assists by: 

extracting structured data 
normalizing addresses, dates, narratives 
identifying missing details 
generating follow-up questions 

This dramatically reduces manual work. 

Now the portal knows the claim is created, but documents still need to be uploaded.

### 4. Customer Uploads Photos → S3 Direct Upload

User uploads:

Damage photo front
Damage photo rear
RC book
Driving license

These go directly to S3, not through the backend.
 

### 5. Initial Claim Record Created 

Stored in OLTP database. 

### 6. Claim Intake Service → Publishes "Claim.Created" Event 

ClaimIntakeCompleted event triggers: 

document ingestion 
enrichment 
triage 
fraud baseline scoring 

### 7. Customer Gets Confirmation 

Claim ID returned, ready for next steps. 

### Tech Used:
1. API Gateway: AWS API Gateway/Kong
2. Frontend: React(Web),Twilio/WhatsApp API for chat
3. LLM: GPT-5/Claude
4. Storage: S3 for media,Postgres for claim metadata.