# Payment Disbursement & Reconciliation

This stage ensures the approved settlement amount is actually transferred to the customer and fully reconciled in financial records. It is one of the most critical parts from regulatory, audit, and customer experience perspectives.

### 1. Trigger: Settlement Completed

Payment Disbursement begins only after the claim is approved.

#### Two possible sources:

#### 1. Auto Settlement Completed

Low-risk, low-amount, high-confidence cases.

#### 2. Human Adjuster Settlement Completed

Complex or high-value claims approved manually.

Event published:

HumanSettlementCompleted {
  "claim_id": "CLM123",
  "approved_amount": 42800,
  "adjuster_id": "ADJ45",
  "timestamp": "2025-11-29T10:30:00"
}

This signal is consumed by the Payment Orchestrator Service, which drives all subsequent steps.

### 2. Fetch Payout Details

The Payment Orchestrator must gather all information required to initiate a financial transfer.

#### Data fetched:

Bank account number
Beneficiary name (for bank account validation / penny drop)
IFSC code
UPI ID (if customer opted for instant payout)
Wallet preferences (Paytm, Amazon Pay—rare for insurance)
Aadhaar-linked account (optional, for direct benefit transfer)
KYC status (mandatory before paying)
Compliance flags (internal or regulatory)

#### Services involved:

Claims Master DB → claimant information
KYC Service → PAN/Aadhaar verification
Banking Microservice → validates account, fetches payout method preferences

Insurance payouts cannot be made without validating who is being paid to avoid fraud, AML violations, and compliance breaches.

### 3. Payment Eligibility Validation

Before money leaves the insurance company’s treasury, the system must ensure the recipient and transaction are legally allowed.

#### Validations:

#### KYC Check

PAN/Aadhaar verified?
Name match between KYC and bank account?

#### AML / Sanction Screening

Ensure claimant isn’t on:
RBI defaulters list
internal blacklist
external AML watchlists

#### Fraud Score Check

Fraud module's output must be below configured threshold.

#### Bank Account Validation

Penny-drop or Bank Account Verification API must confirm:

Account belongs to claimant
IFSC valid
Account active

#### Amount Check

Approved amount must match:

Adjuster-approved settlement
Policy limits

#### Compliance Flags

No blocks from legal, audit, or special investigation units (SIU).

If any check fails →

#### PaymentRejected event → Adjuster notified → Customer asked for corrections

### 4. Payment Initiation (via Bank or Gateway)

Once cleared, the payout is initiated.

Supported payment rails:

#### Bank Transfers

NEFT (batch, slower)
IMPS (near-instant)
RTGS (high-value transfers)

#### Instant Payout APIs

Used for fast, same-minute settlement:
RazorpayX
Cashfree Payouts
Paytm Payouts
PayU
Banks offering API-based IMPS/NEFT

#### International Transfers (optional)

For:

Travel insurance
Overseas medical claims

#### The bank/gateway immediately returns:

transaction ID
status: initiated/pending

### 5. Ledger Posting — The Financial Core

Insurers must maintain double-entry accounting for all payouts.

Required ledger entries:

#### Debit → Claims Reserve Account

Meaning: the reserved liability is reduced.

#### Credit → Customer Payout Account

Meaning: money has been allocated for payout.

#### Metadata stored:

Settlement amount
Claim ID
Ledger ID
Bank transaction ID
Approver / Adjuster ID
Timestamp
Model version & rules used (for audit)

### 6. Bank Confirms Transfer

The bank or payment gateway sends an asynchronous callback.

#### Possible statuses:

Success → funds deposited
Rejected → incorrect account, IFSC mismatch
Pending → bank batch delay
On hold → AML/compliance reason

These confirmations update:

payout table
ledger table
reconciliation queue

#### Idempotent processing:

If bank resends callback →
System should NOT update twice.

Uses:

idempotency_key
provider_txn_id
claim_id

### 7. Reconciliation Engine

This is where financial correctness is guaranteed.

The reconciliation service compares two sources:

A) Internal Ledger

Provisional debit + credit entries
Payout request
Provider transaction ID

B) Bank Statements

Daily statement files
ISO20022 CAMT messages
API-based transaction status
UTR data

#### Output classifications:

#### matched
System and bank aligned. Payout confirmed.

#### unmatched
Payout status in system but not reflecting in bank statement.

#### retry_required
Temporary bank failure → system retries.

#### manual_investigation
Finance team needs to check.

#### Matching logic uses:

UTR number
Transaction timestamp
Amount
Beneficiary details
Transaction ID

### 8. Customer Notification

Once reconciliation or bank success is confirmed, the customer is informed.

Channels:

SMS
→ “Your claim payment of ₹42,800 has been successfully transferred.”

WhatsApp
→ With green tick verified business messaging.

Email
→ Payment advice PDF attached.

### 9. Exceptions & Retry Logic

If a failure occurs:

#### Automatic retries:

If UPI fails → try IMPS
If IMPS fails → try NEFT
If NEFT fails → request updated bank details

#### Manual escalation:

Incorrect bank details
Suspicious patterns
Sanction list conflicts
High-value cases
Repeated failures

#### Operations team is notified through:

Alerts
Dashboard
Email/SMS

### 10. Payment Completed Event

Final event sent to workflow:

PaymentCompleted {
  "claim_id": "CLM123",
  "amount": 42800,
  "txn_id": "IMPS987654321",
  "utr": "HDFC1234567890",
  "timestamp": "2025-11-29T10:45:00"
}
This signals:

Claim is ready for closure
Finance has reconciled it
Customer can be updated

# The claim is now:

Paid
Reconciled
Auditable
Closed
And the customer receives final confirmation.