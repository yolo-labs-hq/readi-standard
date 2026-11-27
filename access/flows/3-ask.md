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
| RA-3.12 (proposed) | A request for read-only access SHOULD use a typed `authorization_details` entry whose actions are all read-only, rather than a bare scope. The approval screen MUST say what can be seen, at which site, and for how long, and MUST NOT describe it as permission to change anything. A read-only grant MUST NOT authorize any action that changes state. | RFC 9396 §§2, 3; READI profile |

**When the sign-in server is split** into an identity provider and an authorization server, the agent still starts here, at the AS. The AS signs the person in through the identity provider, so the person opens one link (spec §8.1).

### Read-only approvals (proposed)

To list or read, ask for a typed read permission rather than a bare scope, so the approval screen can say exactly what the agent will see:

```
&authorization_details=[{"type":"https://readi.page/access/types/booking","actions":["read"],"locations":["https://cafe.example/api/bookings"]}]
```

> **Let BookingBot see your bookings at Café Example?** It can't make, change or cancel a booking. Access ends in 10 minutes.
> [Allow] [Don't allow]

A scope alone, such as `bookings.read`, still works where a site accepts it, but it gives the approval screen less to show.

## Optional: device authorization (proposed)

For a person who is present when the agent can't receive a redirect, such as a command-line agent with no browser callback. The person approves on the server's own page, on any device. If the person may be away, use flow 4 instead.

**1. The agent sends the request on the backchannel** (RFC 8628 §3.1), with the exact resource and details, authenticated as itself:

```http
POST /access/demo/device_authorization HTTP/1.1
Host: readi.page
Content-Type: application/x-www-form-urlencoded

client_id=https%3A%2F%2Fagent.example%2Fagents%2Fassistant.json
&resource=https%3A%2F%2Freadi.page%2Faccess%2Fdemo%2Fmcp
&authorization_details=[{"type":"https://readi.page/access/types/booking","actions":["create"],"locations":["https://readi.page/access/demo/mcp"],"max_party_size":4,"before":"2026-12-05"}]
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&client_assertion=<JWT>
```
```json
{
  "device_code": "<private, kept by the agent>",
  "user_code": "WDJB-MJHT",
  "verification_uri": "https://readi.page/access/demo/device",
  "verification_uri_complete": "https://readi.page/access/demo/device?user_code=WDJB-MJHT",
  "expires_in": 300,
  "interval": 5
}
```

**2. The agent tells the person where to go and what code to expect:**

> Approve this at **readi.page/access/demo/device** with code **WDJB-MJHT**. The code expires at 7:05 pm.

**3. The person signs in on the server's page,** sees the agent (flow 2), the site and each requested action, checks that the code matches the one the agent showed, and approves or declines.

**4. The agent polls the token endpoint** no faster than `interval`, with a DPoP proof, until it gets a token or `access_denied` / `expired_token`:

```http
POST /access/demo/token HTTP/1.1
Host: readi.page
DPoP: <proof>
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:device_code
&device_code=<device code>
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&client_assertion=<JWT>
```

Device authorization doesn't stop someone from tricking a person into approving a code for another person's session. A short code lifetime, attempt limits, showing the expected origin and exact details, and asking the person to check the code each reduce part of that risk. DPoP binds the token to the requesting client but can't correct an approval given to the wrong session.

| ID | Requirement | Source |
|---|---|---|
| RA-3.9 (proposed) | An AS offering device authorization MUST advertise its device authorization endpoint and grant type. The client MUST submit the exact resource and typed details on the device authorization backchannel. The AS MUST bind them and the validated client to the device session and enforce that binding at redemption. RA-3.1, RA-3.2 and RA-2.4 apply to the browser code path. RA-2.6 MUST check response-type intersection only for a grant that uses response types. Device issuance MUST meet flow 5, including DPoP. Any split deployment MUST also meet spec §8. | RFC 8628 §§3, 4; RFC 9396; RFC 8707; RFC 9449; READI profile |
| RA-3.10 (proposed) | The client MUST show the AS verification origin, user code and expiry. The AS MUST show the requesting client, exact resource and approved action details before consent, and ask the person to verify the displayed code against the client even when using `verification_uri_complete`. The AS MUST limit code attempts, expire the code after at most five minutes and allow one redemption. The client MUST keep the device code out of browser URLs, obey `interval` and `slow_down`, and stop on denial or expiry. | RFC 8628 §§3.3, 3.5, 5; RFC 9396 §3; READI profile |

On a split IdP and AS, the AS's verification page signs the person in through the identity provider, as in spec §8.1. The agent still sends the device request to the AS.

## Open question

- Whether READI publishes a small shared registry of plain-language `authorization_details` types (booking, quote request, appointment change, payment), so approval screens say the same thing everywhere.
