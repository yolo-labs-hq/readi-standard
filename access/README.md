# READI Access — Draft 2026.12-pre

*A profile for how AI agents sign in, get a person's approval, and act on their behalf. It is part of the READI readiness standard (https://readi.page), sponsored by The Radar Network. The text is licensed CC BY 4.0. Status: working draft, not for implementation. Revised November 27, 2026: chat platforms and hosted agents (§10), device authorization (flow 3), and platform and agent-host identities. Requirements marked “proposed” are new in this revision and open for comment.*

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
| **Platform** | The product through which the person invokes the agent, for example a chat app. It may only provide the interface. |
| **Agent host** | The service that runs the agent and holds its credentials and keys, for example a hosted integration behind a chat app. |
| **READI Access server** | The authorization server that signs the person in, records approvals and issues tokens. It may be run by the resource or by a broker. It MAY be split into an identity provider and an authorization server (§8). |
| **Upstream provider** | An identity or data provider behind a broker, for example Google. Agents never receive its tokens. |
| **Standing approval** | An approval the person gives once that covers repeated requests from one agent to one resource within exact limits until it expires (§9). |

One organization may fill several roles, for example operator, agent host and platform. Naming a platform or host does not make it a delegating actor: only a verified delegation appears in the token's `act` chain (RA-5.10).

## 3. Standards this profile builds on

| Standard | Used for | Status (checked Nov 24, 2026) |
|---|---|---|
| OAuth 2.1 (draft-ietf-oauth-v2-1) | Core sign-in and token rules, PKCE required | IETF OAuth WG draft -17 (Oct 31, 2026); not yet an RFC |
| RFC 9728, Protected Resource Metadata | Flow 1: finding the sign-in server from a 401 | RFC (published) |
| RFC 8414, Authorization Server Metadata | Flow 1: reading the sign-in server's capabilities | RFC (published) |
| OAuth Client ID Metadata Document (draft-ietf-oauth-client-id-metadata-document) | Flow 2: an agent identified by a URL, with no pre-registration | IETF OAuth WG draft -03, with an IESG review record |
| RFC 9421, HTTP Message Signatures; Web Bot Auth (draft-ietf-webbotauth-httpsig-protocol) | Flow 2 (optional): signed agent requests | RFC 9421 published; Web Bot Auth IETF WG draft -01 (Oct 29, 2026) |
| RFC 9396, Rich Authorization Requests | Flows 3 and 6: exact, readable approvals | RFC (published) |
| RFC 9126, Pushed Authorization Requests | Flow 3: request details sent straight to the server, not through the browser | RFC (published) |
| RFC 8628, Device Authorization Grant | Flow 3 (optional): approval by a present person when the client can't receive a redirect | RFC (published) |
| RFC 9207, Authorization Server Issuer Identification | Flow 1 and §10: checking which server answered | RFC (published) |
| RFC 8707, Resource Indicators | Flows 3 and 5: tokens bound to one site | RFC (published) |
| OpenID CIBA Core 1.0 | Flow 4: approval on the person's phone while the agent waits | OpenID Final Specification |
| RFC 9449, DPoP | Flow 5: tokens usable only by the agent that obtained them | RFC (published) |
| RFC 8693, Token Exchange (`act` claim) | Flow 5: the token names the agent acting for the person | RFC (published) |
| Identity Assertion JWT Authorization Grant (draft-ietf-oauth-identity-assertion-authz-grant) | §8: linking a separate identity provider and authorization server | IETF OAuth WG draft -06; not yet an RFC |
| On-Behalf-Of User Authorization for AI Agents (draft-oauth-ai-agents-on-behalf-of-user) | Flow 5 (watch): agent-specific delegation | Individual draft; not adopted. Informative only |
| AAuth (draft-hardt-aauth-protocol) | Alternative design to track | Individual draft. Informative only |
| RFC 7009, Token Revocation | Flow 7: revoke | RFC (published) |
| RFC 7662, Token Introspection | §11 (experimental bearer class only) | RFC (published) |
| MCP authorization, revision 2026-07-28 | §10: the platform-facing MCP endpoint of a hosted integration | Model Context Protocol specification (industry). Cites OAuth 2.1 draft-13 and CIMD draft-00 |

**Rule:** a normative requirement may rest only on a published RFC, an OpenID Final Specification, or a working-group draft that's explicitly marked as such. Individual drafts are informative. The one exception is the MCP authorization specification, which READI cites only for MCP-facing endpoints (§10) and tests against the pinned revision as written.

**Pinned versions:** OAuth 2.1 draft-17, CIMD draft-03, ID-JAG draft-06 and MCP authorization 2026-07-28. Where MCP cites an older draft than READI pins, the MCP-facing endpoint follows MCP and the READI-protected resource follows READI.

## 4. The seven flows

1. **Discover:** find the sign-in server from a 401 response or a well-known address.
2. **Introduce:** the agent presents a URL identity (Client ID Metadata Document), optionally with signed requests.
3. **Ask (person present):** authorization code with PKCE, an exact approval request (RAR), bound to one resource. Optionally, device authorization (RFC 8628) when the person is present but the client can't receive a redirect.
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

Levels are assessed on the READI-protected resource. In a hosted integration (§10), the platform-facing endpoint is a separate resource and doesn't count toward R4 or R5 by itself.

## 6. Conformance language

The key words MUST, SHOULD and MAY are used as in RFC 2119 and RFC 8174 when in capitals. Each requirement carries an identifier (for example `RA-1.2`), the flow it belongs to, and the standard section it relies on, so a checker can report it in READI's six check states.

## 7. Open questions

- Whether flow 5 should require RFC 8693 token exchange, or allow the Access server to issue the `act` claim directly.
- Whether signed agent requests (Web Bot Auth) become required at R5.
- How a broker shows the person which upstream data (for example Google Calendar) each agent can reach.
- Whether the 90-day ceiling in RA-9.5 should be shorter for payments.
- Whether `readi_standing_approvals` should be proposed to the IETF OAuth working group as a registered metadata member.
- How device authorization (flow 3) bootstraps identity on a split IdP and AS (§8). RFC 8628 defines access-token issuance only, so READI specifies the combined server first.
- How CIBA (flow 4) fits the split exchange and approval in §8. CIBA can return an OpenID ID token, but READI needs a worked example before requiring it.
- What evidence ties a platform's verified service identity to a specific delegation, so it can appear as an inner `act` (RA-5.10).
- Whether Claude's enterprise-managed authorization (which uses ID-JAG in an MCP client) can meet §8 directly. An identity assertion alone still isn't approval (RA-8.3).

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

## 10. Chat platforms and hosted agents (proposed)

*Draft 2026.12-pre, revised November 27, 2026. Text CC BY 4.0.*

Many people will reach an agent through a chat platform rather than an agent they run themselves. Chat platforms implement OAuth for their own connectors, and that OAuth stops short of what READI needs at R4 and R5: DPoP on every call (RA-5.1), exact approvals (flows 3 and 6) and, on a split deployment, ID-JAG (§8). This section says how such a platform can still reach a READI resource without weakening the protected hop.

### 10.1 Hosted access

READI's model for chat platforms is **hosted access**: the platform talks to a hosted integration, and the hosted integration is the READI client.

```
Person → Platform (chat app)
           │  platform OAuth, Bearer (MCP or Action endpoint)
           ▼
         Platform ingress ─ hosted integration (agent host) ─┐
                                                              │  READI flows 1–7: CIMD client, PAR/RAR,
                                                              │  DPoP on every call, ID-JAG under §8
                                                              ▼
                                                         READI resource
```

- **Two resources, two audiences.** The platform-facing endpoint is its own protected resource, with its own metadata and tokens (RA-1.7). The READI resource keeps every flow 5 rule. A token for one is never accepted at the other (RA-5.8).
- **The hosted integration holds the keys.** It is a READI client in its own right: it has a client ID metadata document (flow 2), runs flows 3, 4 and 6, and obtains its own DPoP-bound grant. It never forwards the platform's bearer token downstream (RA-5.8).
- **Platform ingress doesn't approve anything.** A platform sign-in proves which platform account is calling. Approval of exact terms still happens at the READI Access server (flows 3 and 4, RA-8.3).
- **Identities are shown and recorded as verified.** The approval screen names the agent and operator, and the host and platform when they're independently verified (RA-2.9). Tokens and receipts keep that context without inventing a delegation chain (RA-5.10, RA-6.7).

This follows the same token-custody rule as the Google reference design: whoever holds a credential is the only component that uses it.

A person who is present but whose client can't receive a redirect, such as a command-line agent in a sandbox, can use device authorization (flow 3) instead of a callback relay. A hosted integration can receive an ordinary redirect. Neither option gives network access to a sandbox that has none.

### 10.2 What platforms document today

This table summarizes public vendor documentation as of November 27, 2026. READI has not verified any of these products end to end. READI's own test runs so far are one completed booking from a coding agent and one chat-app run that stopped at sign-in.

| Client surface | Documented behavior | Not yet verified for READI |
|---|---|---|
| ChatGPT GPT Actions | OAuth with a configured client ID and secret, authorization code with a platform callback, an authorization header and optional refresh. Bearer or Basic header handling. ([OpenAI: Action authentication](https://developers.openai.com/api/docs/actions/authentication)) | S256 PKCE, OAuth 2.1 conformance, CIMD or dynamic registration, RFC 8707, PAR/RAR, DPoP, ID-JAG |
| ChatGPT apps and MCP connectors | Authorization code with S256, protected-resource discovery, `resource` on authorize and token requests. CIMD with `none` or `private_key_jwt`; dynamic registration and predefined credentials. Sends Bearer, documents refresh-token rotation, and presents an OpenAI-managed TLS client certificate. No JWT-bearer grants. ([OpenAI: app authentication](https://developers.openai.com/plugins/build/auth)) | PAR/RAR, DPoP on every call, restarting an exact approval after a single-use token is spent |
| Claude custom remote connectors | Anthropic infrastructure calls the server. Setup offers Claude's published identity, automatic registration or supplied credentials, with a Claude callback. Enterprise-managed authorization uses ID-JAG in Claude's MCP layer. The Messages API connector takes a caller-supplied token and leaves OAuth to the caller. ([Claude connectors](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp), [Messages API MCP connector](https://platform.claude.com/docs/en/agents-and-tools/mcp-connector), [MCP enterprise-managed authorization](https://blog.modelcontextprotocol.io/posts/enterprise-managed-auth/)) | Exact CIMD and registration behavior, S256 and RFC 8707 on ordinary connectors, refresh across modes, PAR/RAR, DPoP on every call, direct interoperability with §8 |
| Other MCP clients | MCP authorization 2026-07-28 profiles OAuth 2.1 for protected HTTP: RFC 9728 discovery and RFC 8707 resource indicators are required, S256 and Bearer transport are specified, CIMD is recommended, pre-registration is supported, and dynamic registration is optional and deprecated. Refresh is up to the server. READI's flow 1 test reached authorization with CIMD, S256 and the exact resource. ([MCP authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization)) | Any specific product beyond its documentation. Core MCP defines no PAR/RAR, DPoP or ID-JAG contract |

So today a chat platform can reach the platform-facing endpoint of a hosted integration with ordinary MCP or Action OAuth, and the hosted integration does the READI part.

### 10.3 Requirements

The requirements live in the flow pages. This is the index.

| ID | Where | Summary |
|---|---|---|
| RA-1.7 (proposed) | Flow 1 | An MCP-facing endpoint publishes its own metadata, follows MCP resource indicators, and challenges with the scheme it accepts. |
| RA-2.9 (proposed) | Flow 2 | Consent names the agent, operator, and verified host and platform. Unverified platform hints are labelled and never grant access. |
| RA-3.9, RA-3.10 (proposed) | Flow 3 | Optional device authorization, bound to the exact resource and details, with a short code the person checks. |
| RA-5.8 (proposed) | Flow 5 | Separate audiences for platform ingress and READI access. The hosted integration gets its own DPoP-bound grant and never forwards a platform token. |
| RA-5.10 (proposed) | Flow 5 | `sub` is the person, the outer `act` is the current client, and only verified delegating actors nest. Context goes in `readi_context`. |
| RA-6.7 (proposed) | Flows 6 and 7 | Receipts and the review view keep the verified agent, operator, host and platform context and the actor chain. |

## 11. Further proposals (not yet in the flows)

*Draft 2026.12-pre, revised November 27, 2026. Text CC BY 4.0.*

These come from the same review of chat platforms and READI's first live runs. They're published for comment and aren't part of any flow yet.

### Experimental: bearer compatibility class

Some platforms can only send Bearer tokens. Hosted access (§10) is READI's answer. This class is a separately labelled experiment for low-impact operations, to be tested only after connector reauthorization and exact consent are shown to work. A stolen bearer token can still win the race to spend its one use.

| ID | Requirement | Source |
|---|---|---|
| RA-5.9 (proposed, experimental) | A bearer compatibility deployment MUST use a separately advertised resource, explicit issuer, client and resource opt-in, and an opaque token valid for at most two minutes. It MUST issue no refresh token or standing approval. The resource MUST use authenticated, uncached introspection on each attempted use, verify active state, issuer, exact audience, expiry and granted details, and fail closed if validation is unavailable. It MUST atomically limit execution to one exact, freshly approved, low-impact operation. The AS MUST bind those details to the authorization interaction and show them before issuing the token. A scope alone MUST NOT serve as exact approval. This class MUST NOT authorize payments or bookings or count toward R4 or R5. | RFC 6750 §5; RFC 7662 §2; RFC 8707; RFC 9396 §§7, 9; READI profile |

### Software the resource asks the agent to run

| ID | Requirement | Source |
|---|---|---|
| RA-2.10 (proposed) | A deployment directing an agent to execute downloadable software MUST publish a versioned source or release reference and a SHA-256 digest through a publisher channel trusted independently of the download. It SHOULD publish a signed manifest with a verification key that can be authenticated independently. Before execution, the agent MUST verify the exact bytes against the trusted digest or manifest and stop if verification fails. | RFC 7515 §§5, 10; READI profile |

A digest served by the same site as the download only detects accidental corruption. Establishing who published the software needs a key trusted through a separate channel.

### Cross-device handoff and progress

| ID | Requirement | Source |
|---|---|---|
| RA-3.11 (proposed) | The agent SHOULD present a short labelled HTTPS handoff on a known origin and MUST show its server-supplied expiry. A browser code flow SHOULD use PAR to keep request data off the front channel (RA-3.2 still requires it for `authorization_details`). A deployment using a continuation link MUST bind it to the specific client session, enforce its expiry and limit redirects to configured origins. It SHOULD continue from sign-in to consent in one browser handoff, with every required approval still explicit. | RFC 9126 §§2–4; READI profile |
| RA-4.8 (proposed) | A hosted or polling integration MUST expose machine-readable waiting-for-sign-in, waiting-for-approval, approved, denied, expired and failed states as applicable, with a server-supplied expiry for pending interactions. A transaction integration MUST distinguish approved from committed and expose the receipt or its reference when available. Polling MUST require client authentication or a secret separate from the browser link, and the client MUST stop polling at a terminal state. | RFC 8628 §3.5; CIBA §7.4; READI profile |

Progress is exposed through the agent's existing interface, such as command-line JSON lines or tool results. READI defines no new wire protocol for it.

### Receipts that match what happened

| ID | Requirement | Source |
|---|---|---|
| RA-6.8 (proposed; clarifies RA-6.3, RA-6.4 and RA-6.6) | The resource MUST make execution idempotent for the transaction ID and approved terms. A confirmed receipt MUST be based on a durable committed resource outcome. The resource MUST sign that outcome, or an AS permitted by the profile MAY sign after receiving authenticated evidence of it from the resource. The receipt MUST identify the signer and transaction ID as well as the fields in RA-6.4. A retry MUST recover the same outcome and MUST NOT execute again. Pending or uncertain execution MUST remain distinct from confirmed. The AS MUST sign and retain an approval-denial receipt if the resource was never called. | RFC 7515; READI profile |

### An exact booking needs one instant

| ID | Requirement | Source |
|---|---|---|
| RA-6.9 (proposed) | An exact booking detail MUST identify one instant using RFC 3339 with UTC or an explicit known numeric offset. The agent MUST resolve ambiguous dates, zones, and repeated or nonexistent local times before requesting consent. The AS MUST show the date, local time and zone or offset the person is approving. The resource MUST enforce the approved instant, and the receipt MUST preserve it and the zone or offset shown. | RFC 3339 §5.6; RFC 9557 §§2–3; RFC 9396 §2 |

An offset identifies the instant but not what the person meant by “7pm”. Booking types MAY also carry the venue's named zone (RFC 9557) for display.
