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

Accident photos
Videos
CCTV footage (if uploaded)
Workshop inspection photos
EXIF metadata (location, timestamp)
User metadata (upload time, session logs)
Image embeddings (if already generated)

### 3. Vision AI – Damage Classification:

#### Models

YOLOv8 / YOLOv10 – fast bounding-box detection
OpenAI Vision – high-level reasoning
Detectron2 – region-based detection

#### What it detects

Dents
Cracks
Scratches
Broken lights
Deformed panels
Paint peel
Glass shatter
Misalignment

This is the first machine understanding of what is damaged. Later stages take this as input for repair cost estimation

### 4. Part Segmentation (Masking):

Identify exact vehicle parts affected, not just generic “dent”.

#### Models

SAM (Segment Anything Model)
Mask2Former (Transformer-based segmentation)

#### Output

Pixel mask for:
bumper, fender, door, headlight, hood, windshield, etc.

YOLO can say “dent detected”,
Segmentation tells WHERE and WHICH part.

Without this, pricing and garage routing is impossible

### 5. Severity Scoring

#### Convert damage detection into repair complexity:

Minor
Moderate
Severe

#### Input Signals

Depth of dent (estimated from shading + ML)
Crack length
Area of scratch
Mask area (in cm²)
Location importance (structural vs cosmetic)
Historical labeled data (training)

#### Model type

CNN/ViT regression head
Multiclass classifier

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

Labour hours per repair task
Painting cost per panel
Standard PDR costs
Internal OEM part prices (cached)
Tax percentages
Workshop negotiated rates

### 8. Garage API

#### Each garage provides:

Labour rate per hour
Repair time estimate
Booth painting charges
Discount eligibility
Spare availability

### 9. Estimation Engine (Core Component)

#### Inputs

Parts list (segmentation)
Damage type + severity
Action (repair vs replace)
Cost lookup (internal/OEM/garage)
Taxes

#### Process

For each damaged part:

calculate repair + paint cost
calculate replacement cost
apply labour rate
apply paint consumables
Add taxes

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