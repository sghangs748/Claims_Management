# Customer Communication 

This layer ensures that every claim action results in the right message, sent to the right customer, through the right channel, with the right context

### 1. Event-Based Trigger System

Every communication begins with an event coming from the workflow engine or microservices.

Typical events from previous stages

| Stage      | Example Event                           |
| ---------- | --------------------------------------- |
| FNOL       | ClaimReceived, FNOLAcknowledged         |
| Assessment | EstimateReady, InspectionRequired       |
| Fraud      | AdditionalInfoNeeded, FraudCheckPending |
| Settlement | SettlementApproved, SettlementRejected  |
| Payments   | PaymentDisbursed                        |
| Others     | DocumentMissing, CustomerUpdatedPhoto   |