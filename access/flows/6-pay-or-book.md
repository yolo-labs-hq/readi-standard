# READI Access — Flow 6: Pay or book

*Draft 2026.12-pre. How an agent completes a booking or payment for a person, with an exact approval each time and a receipt. Text CC BY 4.0.*

**Goal:** no standing permission to spend. Each transaction gets its own approval with its exact terms (what, when, how much, cancellation terms), and both the person and the business get a signed record of what was agreed.

## Steps

**1. The agent builds a transaction request** as one `authorization_details` entry with every material term:

```json
[{
  "type": "https://readi.page/access/types/booking",
  "actions": ["create"],
  "locations": ["https://readi.page/access/demo/mcp"],
  "item": "Table for 4",
  "start": "2026-12-05T19:00:00-07:00",
  "price": { "amount": "0.00", "currency": "USD" },
  "deposit": { "amount": "20.00", "currency": "USD", "refundable_until": "2026-12-04T19:00:00-07:00" },
  "cancellation": "Free until 24 hours before; deposit kept after that",
  "transaction_id": "txn_7Q2K"
}]
```

**2. It gets a fresh approval for exactly that request,** through flow 3 if the person is present or flow 4 if not. Approval is never reused from an earlier grant.

**3. It redeems a single-use, DPoP-bound token** (flow 5) whose `authorization_details` equal the approved terms, and calls the resource once.

**4. The resource returns a signed receipt** (a JWS, typ `readi-receipt+jwt`) to the agent, and the Access server keeps a copy for the person:

```json
{
  "iss": "https://readi.page/access/demo/mcp",
  "sub": "<person>",
  "act": { "sub": "https://agent.example/agents/assistant.json" },
  "transaction_id": "txn_7Q2K",
  "authorization_details": [ … exactly as approved … ],
  "outcome": "confirmed",
  "reference": "BK-55120",
  "iat": 1764001200
}
```

## Requirements

| ID | Requirement | Source |
|---|---|---|
| RA-6.1 | Every booking or payment MUST have its own approval with all material terms in `authorization_details`: item, time, total price, deposit, refund and cancellation terms. | RFC 9396 §2; READI R5 "material terms before confirmation" |
| RA-6.2 | Transaction approvals MUST be single-use, and MUST NOT be widened later or covered by a standing grant. | READI profile |
| RA-6.3 | Each transaction MUST carry a unique `transaction_id`. The resource MUST treat a repeat with the same ID as the same transaction (no double booking or charge). | READI R5 "duplicate submissions" check |
| RA-6.4 | The resource MUST return a signed receipt naming person, agent, terms, outcome and reference, and the Access server MUST keep it for the person (flow 7). | READI profile; JWS (RFC 7515) |
| RA-6.5 | Payment rails stay out of scope. Where an agent payment mandate system is used (for example Google's Agent Payments Protocol, which uses signed intent, cart and payment mandates), the READI receipt SHOULD reference the mandate ID. | AP2 (industry protocol, v0.x); informative |
| RA-6.6 | A failure or decline MUST produce a receipt with `outcome` `declined` or `failed`, so nothing ends silently. | READI R5 "returns an outcome" |
