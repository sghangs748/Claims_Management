# Document & Evidence Ingestion

1. Files are uploaded to S3 (pre-signed URLs) in claim-intake service.
2. Ingestion service picks S3 events, runs preprocessing (thumbnail, resize, EXIF validation)
3. Malware/Virus Scan:

Prevents malicious uploads from reaching downstream systems. Prevent trojans, ransomware, PDFs with embedded malware

Option can be used:

1. Amazon GuardDuty
2. Amazon Macie
3. ClamAV 
4. third-party security scanning (Symantec, Palo Alto) 

### 4. OCR & Document Understanding pipeline extracts structured fields

OCR Tasks:

1. Extract text
2. Extract structured fields (name, DOB, license number)
3. Detect layout (tables, columns)
4. Detect handwritten fields
5. Generate positional metadata (bounding boxes)

### 5. Document Classification

This identifies what type of document/image the user uploaded

1. RC Book
2. Driving License
3. Invoice
4. Hospital Bill
5. FIR
6. Police Report
7. Damage Photo
8. Handwritten note
9. Additional documents (PAN, Aadhaar, etc.)

### 6. Embedding Generation

Stored in Vector DB (Pinecone, Qdrant, Weaviate). 

Embeddings convert documents, OCR text, images, and extracted metadata into a vector representation that allows the system to:

1. Search semantically
2. Compare similarity
3. Retrieve relevant documents
4. Do cross-document reasoning
5. Detect fraud / duplicates
6. Build claimant profiles
7. Improve LLM understanding
8. Build explainable AI decisions
9. Without embeddings, LLMs and ML pipelines lose context and can’t correlate information efficiently.

### 7. Save Metadata

DB stores: 

1. file type 
2. file path 
3. OCR output 
4. vision output 
5. embedding reference 
6. classification tag 
7. fraud hints 

### 8. Event Routing

FileProcessed → triggers: 

1. Data Enrichment 
2. Triage 
3. Fraud Engine 
4. Damage Assessment 
5. AI Agent workflows 

## Tech Used:

1. OCR: Tesseract/AWS Textract

2. Document Classification:

    1. LLM classification (Cost is more,high latency but no need to train, very accurate)
    2. DistilBERT classifier (cost is less,very fast but need training, not good on photos)
    3. Vision Transformer (ViT) (very accurate but need more training than BERT)

3. Embedding Models: 

OpenAI text-embedding-3-large 

