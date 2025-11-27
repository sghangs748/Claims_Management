# Document & Evidence Ingestion

### 1. Files are uploaded to S3 (pre-signed URLs) in claim-intake service.
### 2. Ingestion service picks S3 events, runs preprocessing (thumbnail, resize, EXIF validation)
### 3. Malware/Virus Scan:

Prevents malicious uploads from reaching downstream systems. Prevent trojans, ransomware, PDFs with embedded malware

Amazon GuardDuty
Amazon Macie
ClamAV 
third-party security scanning (Symantec, Palo Alto) 

### 4. OCR & Document Understanding pipeline extracts structured fields

OCR Tasks:

Extract text
Extract structured fields (name, DOB, license number)
Detect layout (tables, columns)
Detect handwritten fields
Generate positional metadata (bounding boxes)

### 5. Document Classification

This identifies what type of document/image the user uploaded

RC Book
Driving License
Invoice
Hospital Bill
FIR
Police Report
Damage Photo
Handwritten note
Additional documents (PAN, Aadhaar, etc.)

### 6. Embedding Generation

Stored in Vector DB (Pinecone, Qdrant, Weaviate). 

Embeddings convert documents, OCR text, images, and extracted metadata into a vector representation that allows the system to:

Search semantically
Compare similarity
Retrieve relevant documents
Do cross-document reasoning
Detect fraud / duplicates
Build claimant profiles
Improve LLM understanding
Build explainable AI decisions
Without embeddings, LLMs and ML pipelines lose context and can’t correlate information efficiently.

### 7. Save Metadata

DB stores: 

file type 
file path 
OCR output 
vision output 
embedding reference 
classification tag 
fraud hints 

### 8. Event Routing

FileProcessed → triggers: 

Data Enrichment 
Triage 
Fraud Engine 
Damage Assessment 
AI Agent workflows 

## Tech Used:
### 1. OCR: Tesseract/AWS Textract
### 2. Document Classification:

LLM classification (Cost is more,high latency but no need to train, very accurate)
DistilBERT classifier (cost is less,very fast but need training, not good on photos)
Vision Transformer (ViT) (very accurate but need more training than BERT)

### 3. Embedding Models: 

OpenAI text-embedding-3-large 

