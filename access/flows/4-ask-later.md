# READI Access — Flow 4: Ask later (person away)

*Draft 2026.12-pre. How an agent asks for a new approval when the person isn't at a browser. Text CC BY 4.0.*

**Goal:** the agent is mid-task and needs something its first approval didn't cover. The person gets a prompt on their own phone, sees exactly what's asked, and approves or declines. The agent never sees their sign-in.

## Steps

**1. The agent sends a backchannel request** directly to the READI Access server (CIBA §7.1), authenticated as itself and signed:

```http
POST /access/demo/bc-authorize HTTP/1.1
Host: readi.page
Content-Type: application/x-www-form-urlencoded

request=<signed JWT carrying: scope, login_hint_token, binding_message="Book 7pm table for 4 · code K7QD", authorization_details=[…], aud, iss, exp, iat, nbf, jti>
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&client_assertion=<JWT>
```

**2. The server answers with a request ID** (CIBA §7.3):

```json
{ "auth_req_id": "1c266114-a1be-4252-8ad1-04986c5b9ac1", "expires_in": 120, "interval": 5 }
```

**3. The person's phone shows the request:** the agent's identity (flow 2), the action in plain words, and the same short code the agent showed, if any. The person approves or declines.

**4. The agent polls the token endpoint** no faster than `interval`, with `grant_type=urn:openid:params:grant-type:ciba` and the `auth_req_id`, until it gets a token or an error (`authorization_pending`, `slow_down`, `access_denied`, `expired_token`).

## Requirements

| ID | Requirement | Source |
|---|---|---|
| RA-4.1 | The server MUST advertise `backchannel_token_delivery_modes_supported` and MUST support `poll`. `ping` MAY be offered. `push` is NOT RECOMMENDED for agents. | CIBA §4 (Final); READI choice |
| RA-4.2 | The request MUST include exactly one of `login_hint_token`, `id_token_hint` or `login_hint`. READI servers SHOULD accept only `login_hint_token` or `id_token_hint` that they issued, never a bare email or phone typed by the agent. | CIBA §7.1 (one and only one hint); READI choice |
| RA-4.3 | The request MUST be a signed request object (asymmetric signature, `aud` = issuer, `iss` = `client_id`), and the agent MUST authenticate. | CIBA §7.1.1, §7.1 |
| RA-4.4 | Requests MUST carry a `binding_message` describing the action in plain words, short and plain-text. It is shown both by the agent and on the person's device. | CIBA §7.1 (OPTIONAL there; READI makes it MUST) |
| RA-4.5 | Actions MUST be expressed as `authorization_details` (as in flow 3), and the device prompt MUST show them. | RFC 9396; READI profile |
| RA-4.6 | `auth_req_id` MUST have at least 128 bits of entropy. The agent MUST NOT poll faster than `interval` or send overlapping polls. | CIBA §7.3, §7.4 |
| RA-4.7 | A declined or expired request MUST be reported back to the person in the agent's task, not silently retried. | READI profile |
