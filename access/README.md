# READI Access — Draft 2026.12-pre

*A profile for how AI agents sign in, get a person's approval, and act on their behalf. It is part of the READI readiness standard (https://readi.page), sponsored by The Radar Network. The text is licensed CC BY 4.0. Status: working draft, not for implementation.*

## 1. Purpose

READI Access defines how an AI agent finds where to sign in at a website, identifies itself, gets a person's approval for specific actions, and then acts within exactly that approval, with a record the person can review and revoke.

It **invents no new protocol**. It selects existing open standards, pins the options an agent and a site must use, and adds plain-language requirements and tests. It is to agent access what FAPI is to OAuth for banking: a profile, not a replacement.

## 2. Roles

| Role | Meaning |
|---|---|
| **Person** | The human the agent acts for. Only they can grant or revoke approval. |
| **Agent** | Software acting for the person (an assistant, a coding agent, a booking bot). |
| **Agent operator** | The company or individual that runs the agent and publishes its identity. |
| **Resource** | The website or API the agent wants to use (for example, a business's booking endpoint). |
| **READI Access server** | The authorization server that signs the person in, records approvals and issues tokens. It may be run by the resource or by a broker. MAY be split into an identity provider and an authorization server (see §8) |
| **Upstream provider** | An identity or data provider behind a broker, for example Google. Agents never receive its tokens. |
| **Standing approval** | An approval the person gives once that covers repeated requests from one agent to one resource within exact limits until it expires (§9). |

## 3. Standards this profile builds on

| Standard | Used for | Status (checked Nov 24, 2026) |
|---|---|---|
| OAuth 2.1 (draft-ietf-oauth-v2-1) | Core sign-in and token rules, PKCE required | IETF OAuth WG draft -17 (Oct 31, 2026); not yet an RFC |
| RFC 9728, Protected Resource Metadata | Flow 1: finding the sign-in server from a 401 | RFC (published) |
| RFC 8414, Authorization Server Metadata | Flow 1: reading the sign-in server's capabilities | RFC (published) |
| OAuth Client ID Metadata Document (draft-ietf-oauth-client-id-metadata-document) | Flow 2: an agent identified by a URL, with no pre-registration | IETF OAuth WG draft -03, with an IESG review record |
| RFC 9421, HTTP Message Signatures; Web Bot Auth (draft-ietf-webbotauth-httpsig-protocol) | Flow 2 (optional): signed agent requests | RFC 9421 published; Web Bot Auth IETF WG draft -01 (Oct 29, 2026) |
| RFC 9396, Rich Authorization Requests | Flows 3 and 6: exact, readable approvals | RFC (published) |
| RFC 8707, Resource Indicators | Flows 3 and 5: tokens bound to one site | RFC (published) |
| OpenID CIBA Core 1.0 | Flow 4: approval on the person's phone while the agent waits | OpenID Final Specification |
| RFC 9449, DPoP | Flow 5: tokens usable only by the agent that obtained them | RFC (published) |
| RFC 8693, Token Exchange (`act` claim) | Flow 5: the token names the agent acting for the person | RFC (published) |
| Identity Assertion JWT Authorization Grant (draft-ietf-oauth-identity-assertion-authz-grant) | §8: linking a separate identity provider and authorization server | IETF OAuth WG draft -06; not yet an RFC |
| On-Behalf-Of User Authorization for AI Agents (draft-oauth-ai-agents-on-behalf-of-user) | Flow 5 (watch): agent-specific delegation | Individual draft; not adopted. Informative only |
| AAuth (draft-hardt-aauth-protocol) | Alternative design to track | Individual draft. Informative only |
| RFC 7009, Token Revocation | Flow 7: revoke | RFC (published) |

**Rule:** a normative requirement may rest only on a published RFC, an OpenID Final Specification, or a working-group draft that's explicitly marked as such. Individual drafts are informative.

## 4. The seven flows

1. **Discover:** find the sign-in server from a 401 response or a well-known address.
2. **Introduce:** the agent presents a URL identity (Client ID Metadata Document), optionally with signed requests.
3. **Ask (person present):** authorization code with PKCE, an exact approval request (RAR), bound to one resource.
4. **Ask later (person away):** a CIBA request approved on the person's own device.
5. **Act:** short-lived, sender-constrained (DPoP), resource-bound tokens naming agent and person.
6. **Pay or book:** a fresh, exact approval for each transaction and a signed receipt. Under §9 bounded mode, a standing approval may cover a payment or booking within its caps; every such use still produces a signed receipt.
7. **Review and revoke:** what the person can see and stop at any time. Lists every standing approval (§9) with one-step revoke.

Each flow has its own agent-readable page at `/access/flows/<n>-<name>.md` with exact HTTP requests and responses.

## 5. Mapping to READI levels

| READI level | READI Access requirement |
|---|---|
| R3 Usable | None. A contact path suffices. |
| R4 Actionable | Flows 1–3 and 5 for at least one documented action, and flow 7 revocation. Standing approvals allowed in strict mode only. |
| R5 Transactable | R4 plus flow 6 for every payment or booking, and flow 4 where the person may be away. Bounded mode allowed if RA-9.4 and RA-9.9 are met. |

## 6. Conformance language

The key words MUST, SHOULD and MAY are used as in RFC 2119 and RFC 8174 when in capitals. Each requirement carries an identifier (for example `RA-1.2`), the flow it belongs to, and the standard section it relies on, so a checker can report it in READI's six check states.

## 7. Open questions

- Whether flow 5 should require RFC 8693 token exchange, or allow the Access server to issue the `act` claim directly.
- Whether signed agent requests (Web Bot Auth) become required at R5.
- How a broker shows the person which upstream data (for example Google Calendar) each agent can reach.
- Whether the 90-day ceiling in RA-9.5 should be shorter for payments.
- Whether `readi_standing_approvals` should be proposed to the IETF OAuth working group as a registered metadata member.

## 8. Separate identity provider and authorization server (optional)

*Draft 2026.12-pre. Text CC BY 4.0.*

A deployment MAY run authentication and authorization as two services:
- an **identity provider (IdP)** that signs the person in and vouches for who they are, and
- an **authorization server (AS)** that decides what an agent may do and issues access tokens.

When they are separate, they MUST be linked with the **Identity Assertion JWT Authorization Grant (ID-JAG)** (draft-ietf-oauth-identity-assertion-authz-grant-06, an IETF OAuth working-group draft). Track changes until it is final.

### Steps

**1. The person signs in at the IdP** (OpenID Connect). The agent receives an ID token or refresh token for that person.

**2. The agent exchanges it at the IdP for an ID-JAG** (RFC 8693 token exchange):

```http
POST /oauth2/token HTTP/1.1
Host: idp.example
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&requested_token_type=urn:ietf:params:oauth:token-type:id-jag
&audience=https://as.example/
&resource=https://api.example/
&scope=booking.create
&subject_token=<ID token>
&subject_token_type=urn:ietf:params:oauth:token-type:id_token
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
&client_assertion=<agent's signed JWT>
```

The IdP returns a signed JWT with header `typ: oauth-id-jag+jwt` and claims including `iss`, `sub`, `aud` (the AS), `client_id` (the agent at the AS), `exp`, `iat`, `jti`, `resource` and `scope` (or `authorization_details`).

**3. The agent presents the ID-JAG at the AS** (RFC 7523 JWT bearer grant), with a DPoP proof:

```http
POST /token HTTP/1.1
Host: as.example
DPoP: <proof>
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
&assertion=<ID-JAG>
```

**4. The AS applies its own policy** and either issues a DPoP-bound access token (flow 5) or requires the person's approval first (flows 3–4).

### Requirements

| ID | Requirement | Source |
|---|---|---|
| RA-8.1 | If the IdP and AS are separate services, the AS MUST accept identity only through an ID-JAG, never through a raw ID token sent to the AS. | ID-JAG draft-06 §4 (WG draft) |
| RA-8.2 | The AS MUST validate the ID-JAG: `typ` is `oauth-id-jag+jwt`, the signature is from a trusted IdP, `aud` is the AS, `client_id` matches the presenting agent, and it is unexpired and not replayed (`jti`). | ID-JAG draft-06 §3.1, §4.4; RFC 8725 |
| RA-8.3 | **An ID-JAG proves who the person and agent are. It is not the person's approval.** The AS MUST still obtain approval through flow 3 or 4 for any action the person has not already approved at the AS, and MUST NOT treat IdP-side scopes as consent for transactions (flow 6). | READI profile; ID-JAG draft-06 (the resource AS keeps independent control of its tokens) |
| RA-8.4 | Requested actions SHOULD be carried as `authorization_details` in the token exchange and preserved into the AS's grant. | ID-JAG draft-06 (RAR example); RFC 9396 |
| RA-8.5 | Access tokens issued from an ID-JAG MUST meet flow 5 (DPoP-bound, resource-bound, `act` claim naming the agent). | RFC 9449; RFC 8707; RFC 8693 §4.1 |
| RA-8.6 | Both services MUST publish RFC 8414 metadata, including the ID-JAG-related metadata the draft defines for the IdP and the AS. | ID-JAG draft-06 (IdP and Resource AS metadata); RFC 8414 |
| RA-8.7 | When the IdP issues the agent a long-lived token to obtain ID-JAGs, that token MUST be sender-constrained (DPoP-bound refresh token). | RFC 9449 §5 |
| RA-8.8 | An ID-JAG MUST NOT be valid for more than 5 minutes. | ID-JAG draft-06 (short-lived assertions); READI profile |
| RA-8.9 | When the person signs the agent out at the IdP, the IdP MUST stop issuing ID-JAGs for that agent immediately. | READI profile |
| RA-8.10 | An IdP MAY restrict which authorization servers it issues ID-JAGs for (for example, only one). A resource MUST accept access tokens only from the authorization servers listed in its metadata. | RFC 9728; READI profile |

The spec's roles table should note that the "READI Access server" MAY be split into an identity provider and an authorization server under this section.

## 9. Standing approvals (optional)

*Draft 2026.12-pre. Text CC BY 4.0.*

A **standing approval** is an approval the person gives once that covers repeated requests from one agent to one resource, inside exact limits, until it expires. It lets the agent act again without asking each time, while the person keeps a clear list of what each agent may do and can stop any of it.

A standing approval is a stored grant at the authorization server (AS). It is not a token, and the agent never holds it. Each time the agent acts, the AS checks the request against the stored grant and, if it fits, issues a normal flow 5 access token.

### 9.1 Two modes

| Mode | What a standing approval may cover | Payments and bookings |
|---|---|---|
| **strict** (default) | Read, check availability, place a hold that expires without charge, and other actions that create no obligation for the person | Never covered. Each one needs a fresh, exact approval (flow 6). |
| **bounded** | Everything in strict, plus payments and bookings inside published caps | Covered only within the grant's caps and expiry, with a signed receipt and a notice to the person for every use |

The AS MUST apply the **stricter** of two settings:
- the **resource policy** published in the resource's metadata (9.3), and
- the **person's preference** set at the AS ("Always ask me before paying or booking" = strict; "Allow capped approvals" = bounded).

If either is strict, the mode is strict. If the resource publishes no policy, the mode is strict. If the resource publishes `none`, standing approvals MUST NOT be used for it at all.

### 9.2 What a standing approval contains

A standing approval MUST record:

| Field | Meaning |
|---|---|
| `id` | Unique identifier at the AS |
| `person` | The person who approved it (`sub` at the AS) |
| `agent` | The agent's `client_id` (its Client ID Metadata Document URL) |
| `resource` | The one resource it applies to (RFC 8707 resource indicator) |
| `authorization_details` | The exact actions and limits (RFC 9396), for example `{"type":"booking","actions":["create"],"party_size_max":4,"locations":["https://cafe.example/"]}` |
| `mode` | `strict` or `bounded`, as resolved in 9.1 when the grant was created |
| `caps` | For bounded only: per-use maximum amount and currency, maximum count per day, and any other numeric limits the resource publishes |
| `created_at`, `expires_at` | When it was approved and when it ends |
| `approval_text` | The plain-language sentence the person saw and approved, stored verbatim |
| `last_used_at`, `use_count` | Updated on every use |
| `status` | `active`, `expired` or `revoked` |

### 9.3 Resource metadata

A resource that allows standing approvals MUST publish its policy in its Protected Resource Metadata (RFC 9728) under the READI extension member `readi_standing_approvals`:

```json
{
  "resource": "https://cafe.example/",
  "authorization_servers": ["https://auth.example/"],
  "readi_standing_approvals": {
    "policy": "bounded",
    "max_expiry_days": 30,
    "caps": {
      "booking": { "party_size_max": 6, "per_day_max": 2 },
      "payment": { "amount_max": "50.00", "currency": "USD", "per_day_max": 2 }
    }
  }
}
```

`policy` is one of `none`, `strict` or `bounded`. `max_expiry_days` and `caps` are REQUIRED when `policy` is `bounded`. This member is a READI extension; no IETF or OpenID standard defines it.

### 9.4 Creating a standing approval

The agent asks for a standing approval during flow 3 or flow 4 by adding `"readi_standing": true` and a requested expiry to the `authorization_details` entry. The AS MUST show the person a plain-language sentence that states the agent, the resource, the actions, every limit and the end date, for example:

> **Okay for BookingBot to book tables at Café Example for up to 4 people, at most 2 a day, until December 31, 2026?**
> [Okay] [Only this once] [Not now]

"Only this once" grants the single request without storing a standing approval.

### 9.5 Using a standing approval

When the agent requests a token (flow 3, flow 4, or §8 with an ID-JAG), the AS MUST:
1. find an `active`, unexpired standing approval with the same `person`, `agent` and `resource`;
2. check that the requested `authorization_details` are entirely inside the stored ones (same type and action, every numeric value within its limit);
3. under bounded, check that the request is within every cap, including per-day counts;
4. if all checks pass, issue a flow 5 access token limited to the requested details, with a lifetime of at most 10 minutes; otherwise fall back to asking the person (flow 3 or 4).

### 9.6 Requirements

| ID | Requirement | Source |
|---|---|---|
| RA-9.1 | A standing approval MUST contain the fields in 9.2, and the AS MUST store the approved sentence verbatim. | READI profile; RFC 9396 |
| RA-9.2 | The AS MUST resolve the mode as the stricter of the resource policy and the person's preference. Absent a resource policy, the mode is strict. | READI profile |
| RA-9.3 | Under strict, a standing approval MUST NOT cover any payment, booking, or other action that creates an obligation for the person. Flow 6 applies to each such action. | READI profile |
| RA-9.4 | Under bounded, each use MUST stay within the grant's caps and expiry, MUST produce a signed flow 6 receipt, and MUST notify the person within 5 minutes. | READI profile |
| RA-9.5 | `expires_at` MUST NOT be later than the resource's `max_expiry_days`, and in any case MUST NOT be more than 90 days after `created_at`. | READI profile |
| RA-9.6 | The AS MUST issue tokens from a standing approval only for requests entirely inside its `authorization_details`. Any request outside it MUST go back to the person (flow 3 or 4). | READI profile; RFC 9396 |
| RA-9.7 | Tokens issued from a standing approval MUST meet flow 5 (DPoP-bound, resource-bound, `act` claim) and MUST NOT live longer than 10 minutes. | RFC 9449; RFC 8707; RFC 8693 |
| RA-9.8 | Every standing approval MUST appear in flow 7 with its sentence, limits, mode, last use and expiry, and a one-step revoke. Revoking MUST stop new tokens immediately and SHOULD revoke outstanding ones (RFC 7009). | RFC 7009; READI profile |
| RA-9.9 | A resource advertising `bounded` MUST publish `max_expiry_days` and `caps` (9.3), and MUST enforce the same limits itself when it receives a token. | RFC 9728; READI profile |
| RA-9.10 | The person MUST be able to switch their preference to strict at any time. Switching MUST immediately limit every existing bounded approval to strict behavior. | READI profile |

### 9.7 Tests

A conforming deployment passes these checks:
1. Request inside a strict standing approval (check availability) → token issued without asking.
2. Booking request under strict → sent to the person for a fresh approval.
3. Booking under bounded, within caps → token issued, signed receipt created, notice sent.
4. Booking under bounded with party size above the cap → sent to the person.
5. Third booking in a day with `per_day_max: 2` → sent to the person.
6. Request after `expires_at` → sent to the person, and the grant shows `expired`.
7. Resource publishes `strict`, person prefers bounded → strict behavior.
8. Revoke in flow 7, then request → refused, and any outstanding token is rejected by the resource.
9. Person switches to strict → an existing bounded approval no longer covers bookings.
10. Request for a different resource than the grant's → sent to the person.
