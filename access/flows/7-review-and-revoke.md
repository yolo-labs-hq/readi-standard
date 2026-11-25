# READI Access — Flow 7: Review and revoke

*What the person can always see and stop.*

**Goal:** at any time, the person can see which agents can do what, where, until when, and what each did. One tap ends any of it, and the resource stops honoring it within minutes, not hours.

## What the person's dashboard MUST show

For each agent (by name, logo and `client_id` hostname):
- **Active approvals:** each `authorization_details` entry in plain words, the resource, when it was granted and when it expires.
- **Pending requests:** flow 4 asks waiting for an answer.
- **Activity:** receipts from flow 6 and a log of calls, with time, resource and action.
- **Controls:** "Revoke this approval", "Revoke everything for this agent", and "Revoke everything at this site".

## Steps

**1. The person taps Revoke.** The Access server marks the grant revoked and revokes its tokens (RFC 7009).

**2. The Access server tells resources at once,** using a Shared Signals stream with a CAEP `session-revoked` event, so resources don't have to wait for tokens to expire.

**3. The agent learns on its next call** (401 with `error="invalid_token"`) and reports to the person that the approval was withdrawn.

## Requirements

| ID | Requirement | Source |
|---|---|---|
| RA-7.1 | The Access server MUST offer a revocation endpoint. Agents MUST revoke refresh tokens they no longer need. | RFC 7009 §2 |
| RA-7.2 | The dashboard MUST list every active grant per agent with plain-language details, expiry and activity, and MUST offer per-grant, per-agent and per-site revocation. | READI profile; OpenID Grant Management for OAuth 2.0 (Implementer's Draft 2) informs the grant model |
| RA-7.3 | Revocation MUST take effect at the Access server immediately. Resources MUST stop honoring revoked grants within 5 minutes, either through a Shared Signals / CAEP stream (SHOULD) or by token lifetime (access tokens ≤ 10 minutes per RA-3.8). | OpenID SSF 1.0 and CAEP 1.0 (Final Specifications, approved Sept 2, 2025); READI profile |
| RA-7.4 | Receipts and the activity log MUST be kept for the person for at least 12 months and be exportable. | READI profile |
| RA-7.5 | An agent MUST tell the person when an approval it relied on was revoked or expired, rather than retrying another way. | READI profile |
