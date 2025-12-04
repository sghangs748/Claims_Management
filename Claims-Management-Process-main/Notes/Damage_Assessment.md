# Damage Assessment 

### 1. Trigger From Fraud Check:

Damage assessment consumes heavy GPU resources (YOLO, segmentation, LLM reasoning).
We never want to waste compute on a claim that is already flagged as suspicious

If FraudCheckCompleted.pass == true 
    → trigger DamageAssessmentPipeline
Else
    → route to manual review / fraud team

### 2. Media Fetching Layer:

#### What is fetched

1. Accident photos
2. Videos
3. CCTV footage (if uploaded)
4. Workshop inspection photos
5. EXIF metadata (location, timestamp)
6. User metadata (upload time, session logs)
7. Image embeddings (if already generated)

### 3. Vision AI – Damage Classification:

#### Models

1. YOLOv8 / YOLOv10 – fast bounding-box detection
2. OpenAI Vision – high-level reasoning
3. Detectron2 – region-based detection

#### What it detects

1. Dents
2. Cracks
3. Scratches
4. Broken lights
5. Deformed panels
6. Paint peel
7. Glass shatter
8. Misalignment

This is the first machine understanding of what is damaged. Later stages take this as input for repair cost estimation

### 4. Part Segmentation (Masking):

Identify exact vehicle parts affected, not just generic “dent”.

#### Models

1. SAM (Segment Anything Model)
2. Mask2Former (Transformer-based segmentation)

#### Output

Pixel mask for:
bumper, fender, door, headlight, hood, windshield, etc.

YOLO can say “dent detected”,
Segmentation tells WHERE and WHICH part.

Without this, pricing and garage routing is impossible

### 5. Severity Scoring

#### Convert damage detection into repair complexity:

1. Minor
2. Moderate
3. Severe

#### Input Signals

1. Depth of dent (estimated from shading + ML)
2. Crack length
3. Area of scratch
4. Mask area (in cm²)
5. Location importance (structural vs cosmetic)
6. Historical labeled data (training)

#### Model type

1. CNN/ViT regression head
2. Multiclass classifier

### 6. Action Mapping

Convert severity → repair action

| Damage             | Severity | Action                      |
| ------------------ | -------- | --------------------------- |
| Dent               | Minor    | Paintless Dent Repair (PDR) |
| Dent               | Moderate | Body repair + repaint       |
| Scratch            | Minor    | Buffing                     |
| Broken headlight   | Any      | Replace                     |
| Cracked windshield | Any      | Replace                     |

### 7. Internal Pricing DB Lookup:

#### Pull internal cost data:

1. Labour hours per repair task
2. Painting cost per panel
3. Standard PDR costs
4. Internal OEM part prices (cached)
5. Tax percentages
6. Workshop negotiated rates

### 8. Garage API

#### Each garage provides:

1. Labour rate per hour
2. Repair time estimate
3. Booth painting charges
4. Discount eligibility
5. Spare availability

### 9. Estimation Engine (Core Component)

#### Inputs

1. Parts list (segmentation)
2. Damage type + severity
3. Action (repair vs replace)
4. Cost lookup (internal/OEM/garage)
5. Taxes

#### Process

For each damaged part:

1. calculate repair + paint cost
2. calculate replacement cost
3. apply labour rate
4. apply paint consumables
5. Add taxes

Generate itemized bill

### 10. Final Output Event

DamageAssessmentCompleted {
   claim_id,
   parts,
   labour,
   total_estimate,
   replaced_parts,
   confidence
}