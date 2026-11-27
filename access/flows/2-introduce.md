# READI Access — Flow 2: Introduce

*Draft 2026.12-pre. How an agent tells a site who it is, with no prior registration. Text CC BY 4.0.*

**Goal:** the sign-in server learns which agent is asking, from a URL the agent operator controls, and the person sees that identity before approving anything.

## Steps

**1. The agent operator publishes a Client ID Metadata Document** at an HTTPS URL with a path. That URL *is* the agent's `client_id`.

```http
GET /agents/assistant.json HTTP/1.1
Host: agent.example
Accept: application/json
```
```json
{
  "client_id": "https://agent.example/agents/assistant.json",
  "client_name": "Example Assistant",
  "client_uri": "https://agent.example",
  "logo_uri": "https://agent.example/logo.png",
  "redirect_uris": ["https://agent.example/oauth/callback"],
  "grant_types": ["authorization_code"],
  "response_types": ["code"],
  "token_endpoint_auth_method": "private_key_jwt",
  "jwks_uri": "https://agent.example/.well-known/jwks.json"
}
```

**2. The agent starts authorization using that URL as `client_id`** (this begins flow 3).

**3. The sign-in server fetches and checks the document** before showing anything to the person.

**4. Optional: the agent signs its HTTP requests (Web Bot Auth),** so a website can recognize the agent operator even outside sign-in:

```http
GET /access/demo/mcp HTTP/1.1
Host: readi.page
Signature-Agent: sig1="https://agent.example/.well-known/http-message-signatures-directory"
Signature-Input: sig1=("@authority" "signature-agent";key="sig1");created=1764000000;expires=1764003600;keyid="<JWK SHA-256 thumbprint>";tag="web-bot-auth"
Signature: sig1=:<signature>:
```

## Requirements

| ID | Requirement | Source |
|---|---|---|
| RA-2.1 | The agent's `client_id` MUST be an `https` URL with a path other than `/`, no userinfo, no fragment and no dot segments. It SHOULD NOT have a query. | CIMD draft-03 §3 (WG draft) |
| RA-2.2 | The document's `client_id` MUST exactly equal the URL it was fetched from (simple string comparison). | CIMD §4 |
| RA-2.3 | The document MUST NOT contain `client_secret` or private keys. Agents that authenticate SHOULD use `private_key_jwt` with keys at `jwks_uri`. | CIMD §4.1, §5 |
| RA-2.4 | The sign-in server MUST require registered `redirect_uris` and an exact match, except loopback redirects with random ports. | CIMD §4.2; RFC 9700 §4.1.3 |
| RA-2.5 | The sign-in server MUST accept only a 200 response, MUST NOT follow redirects when fetching the document, and MUST NOT fetch URLs that resolve to special-use IP addresses (no loopback exception in production). | CIMD §6, §10.7 |
| RA-2.6 | The sign-in server MUST intersect the document's grant and response types with its own, and reject with `invalid_client` if either is empty. | CIMD §6 |
| RA-2.7 | The approval screen MUST show the `client_id` hostname, plus fetched name and logo if available. | CIMD §10.6 (SHOULD there; READI makes it MUST) |
| RA-2.8 | OPTIONAL at R4, *under review for R5*: signed requests per Web Bot Auth. If used: a `Signature-Agent` header in dictionary form with an `https` URL, signature covering `@authority` or `@target-uri`, `created`, `expires` (RECOMMENDED ≤ 24 h), a JWK-thumbprint `keyid`, and `tag="web-bot-auth"`. | draft-ietf-webbotauth-httpsig-protocol-01 §5 (WG draft); RFC 9421 |
| RA-2.9 (proposed) | Consent MUST name the agent and its client ID origin, identify the accountable operator, and identify the agent host and originating platform when independently verified and distinct. The AS MUST bind verified identities to the approval using authenticated client evidence or trusted configuration for the specific integration. A request-supplied platform hint MAY be displayed as unverified context and MUST NOT be treated as an authenticated actor or used to grant access. | CIMD draft-03 (WG draft); RFC 8693 §4.1; READI profile |

### What the approval screen shows for a hosted agent

When the agent runs on a hosted integration behind a chat platform (spec §10), the screen separates what's verified from what isn't:

> **Example Assistant** (agent.example) wants to book a table at Café Example.
> Run by Example Labs, hosted by agent.example. Requested from Example Chat (verified).

A platform name taken only from the request is shown as “says it was requested from Example Chat (not verified)”, and never changes what the agent may do.
