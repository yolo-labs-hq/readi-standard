# READI Access — Flow 5: Act

*How an agent uses an approval: tokens that work at one site, only for the agent that got them, and that name both agent and person.*

## Steps

**1. The agent creates a key pair** and sends a DPoP proof with its token request (RFC 9449 §4–5). The token comes back as `"token_type": "DPoP"`, bound to that key.

**2. On every call to the resource, the agent sends the token plus a fresh proof:**

```http
POST /access/demo/mcp HTTP/1.1
Host: readi.page
Authorization: DPoP <access token>
DPoP: <JWT: header {typ:"dpop+jwt", alg, jwk}; claims {jti, htm:"POST", htu:"https://readi.page/access/demo/mcp", iat, ath:<hash of access token>, nonce?}>
```

**3. The token names both parties** (JWT access token claims):

```json
{
  "iss": "https://readi.page/access/demo",
  "aud": "https://readi.page/access/demo/mcp",
  "sub": "<person>",
  "client_id": "https://agent.example/agents/assistant.json",
  "act": { "sub": "https://agent.example/agents/assistant.json" },
  "cnf": { "jkt": "<DPoP key thumbprint>" },
  "authorization_details": [ … as granted … ],
  "exp": 1764000600
}
```

## Requirements

| ID | Requirement | Source |
|---|---|---|
| RA-5.1 | Access tokens MUST be DPoP-bound (`token_type` `DPoP`, `cnf.jkt`). Bearer tokens MUST NOT be accepted by an R4 resource. | RFC 9449 §5, §6 |
| RA-5.2 | Each request MUST carry a new DPoP proof with `typ` `dpop+jwt`, `jti`, `htm`, `htu`, `iat` and `ath`. The resource MUST check all of them and reject reused `jti`s within its window. | RFC 9449 §4.2, §4.3, §7 |
| RA-5.3 | Servers SHOULD use server-provided nonces (`DPoP-Nonce`) to limit pre-generated proofs. | RFC 9449 §8, §9 |
| RA-5.4 | The token's `aud` MUST be the single resource from flow 3. The resource MUST reject tokens for any other audience. | RFC 8707; RFC 9068 |
| RA-5.5 | The token MUST name the person in `sub` and the agent in an `act` claim. If the agent acts through another agent, `act` claims nest, newest first (see RA-5.10). | RFC 8693 §4.1 |
| RA-5.6 | The resource MUST enforce the granted `authorization_details`, not just the scope. | RFC 9396 §9 |
| RA-5.7 | Refresh tokens, if issued, MUST be DPoP-bound or rotated, and MUST NOT outlive the person's approval. | OAuth 2.1 draft-17 (WG draft); RFC 9449 §5 |
| RA-5.8 (proposed) | A hosted integration MUST use distinct audiences for platform ingress and downstream READI access. It MUST keep READI and upstream credentials and keys within the component authorized to hold them. It MUST obtain its own downstream DPoP-bound grant and MUST follow spec §8 on a split READI deployment. It MUST NOT forward a platform bearer token as a downstream credential. Platform ingress MUST NOT authorize a downstream transaction by itself. | RFC 8707; RFC 9449; spec §8 |
| RA-5.10 (proposed) | The top-level `sub` MUST identify the person. The outer `act` MUST identify the current acting client. Only verified prior delegating actors MAY appear in nested `act` objects, most recent first, with issuer context where needed to disambiguate identifiers. The resource MUST base authorization on the token's top-level claims and current actor. A verified host or platform that supplies context only MAY appear in separate READI profile fields (`readi_context`). | RFC 8693 §4.1 |

## Hosted agents: who goes in `act` (proposed)

When a chat platform reaches a READI resource through a hosted integration (spec §10), the hosted integration is the READI client, so it is the outer `act`. The platform appears as an inner `act` only if it actually delegated to the hosted client and the deployment has evidence tying the platform to that delegation. A platform that only provides the chat interface, or a host that only runs the code, is recorded in `readi_context` instead:

```json
{
  "iss": "https://readi.page/access/demo",
  "aud": "https://readi.page/access/demo/mcp",
  "sub": "<person>",
  "client_id": "https://agent.example/agents/hosted.json",
  "act": { "sub": "https://agent.example/agents/hosted.json" },
  "readi_context": {
    "operator": { "name": "Example Labs", "uri": "https://agent.example" },
    "host": { "uri": "https://agent.example", "verified": true },
    "platform": { "name": "Example Chat", "uri": "https://chat.example", "verified": true }
  },
  "cnf": { "jkt": "<DPoP key thumbprint>" },
  "authorization_details": [ … as granted … ],
  "exp": 1764000600
}
```

Nested `act` entries are informational for access control (RFC 8693 §4.1): the resource decides on `sub`, the outer `act`, the audience and the granted details. A platform's TLS client certificate can verify the platform service at ingress, but that alone doesn't show it delegated this request. Token exchange is one way to establish a real chain; a server MAY also issue the claim directly from verified context.

The platform-facing endpoint may accept Bearer tokens as MCP specifies (flow 1, RA-1.7). It is a separate resource, so RA-5.1 still applies in full to the READI resource behind it. RA-5.9, a proposed experimental bearer class for low-impact operations only, is in spec §11.

**Watch (informative):** the individual draft "On-Behalf-Of User Authorization for AI Agents" proposes agent-specific parameters (`requested_actor`, `actor_token`) for the same goal. AAuth proposes signed requests instead of tokens. Neither is required. READI tracks both and will align if one is adopted by a working group.
