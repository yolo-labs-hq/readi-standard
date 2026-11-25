# READI Access — Flow 3: Ask (person present)

*How an agent gets a person's approval for specific actions while the person is at a browser.*

**Goal:** the person sees, in plain words, exactly what this agent may do at this one site, approves some or all of it, and the resulting token carries only that.

## Steps

**1. The agent pushes the request to the server first** (Pushed Authorization Request), so the details can't be altered on the way through the browser:

```http
POST /access/demo/par HTTP/1.1
Host: readi.page
Content-Type: application/x-www-form-urlencoded

client_id=https%3A%2F%2Fagent.example%2Fagents%2Fassistant.json
&response_type=code
&redirect_uri=https%3A%2F%2Fagent.example%2Foauth%2Fcallback
&code_challenge=<S256 challenge>&code_challenge_method=S256
&resource=https%3A%2F%2Freadi.page%2Faccess%2Fdemo%2Fmcp
&authorization_details=[{"type":"https://readi.page/access/types/booking","actions":["create"],"locations":["https://readi.page/access/demo/mcp"],"max_party_size":4,"before":"2026-12-05"}]
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&client_assertion=<JWT>
```
→ `201` with a `request_uri`.

**2. The browser goes to the authorize page with only `client_id` and `request_uri`.** The person sees the agent's identity (flow 2) and each requested permission in plain language, and can approve all or part of it.

**3. The agent redeems the code** at the token endpoint, with the PKCE verifier and the same `resource`. The response lists exactly what was granted:

```json
{
  "access_token": "…",
  "token_type": "DPoP",
  "expires_in": 600,
  "authorization_details": [{"type":"https://readi.page/access/types/booking","actions":["create"],"locations":["https://readi.page/access/demo/mcp"],"max_party_size":4,"before":"2026-12-05"}]
}
```

## Requirements

| ID | Requirement | Source |
|---|---|---|
| RA-3.1 | Authorization code flow with PKCE `S256` only. | OAuth 2.1 draft-17 (WG draft) |
| RA-3.2 | Requests carrying `authorization_details` MUST use Pushed Authorization Requests, because RAR data sent through the browser can be modified. | RFC 9126; RFC 9396 §12 |
| RA-3.3 | Each action MUST be expressed as an `authorization_details` object with a `type`. Types MUST be published at a stable URL with a plain-language description the approval screen can show. | RFC 9396 §2, §2.1 |
| RA-3.4 | The server MUST reject unknown or malformed types with `invalid_authorization_details`. | RFC 9396 §5 |
| RA-3.5 | The approval screen MUST show the merged set of what's requested (scopes plus details). The person MAY approve a subset. | RFC 9396 §3, §3.1 |
| RA-3.6 | The token response MUST return the `authorization_details` actually granted. | RFC 9396 §7 |
| RA-3.7 | The authorization and token requests MUST carry `resource`. The token MUST be usable only at that resource. | RFC 8707; RFC 9396 §3.2 |
| RA-3.8 | Access tokens MUST be short-lived (READI default ≤ 10 minutes) and sender-constrained (flow 5). | READI profile choice; RFC 9449 |

## Open question

- Whether READI publishes a small shared registry of plain-language `authorization_details` types (booking, quote request, appointment change, payment), so approval screens say the same thing everywhere.
