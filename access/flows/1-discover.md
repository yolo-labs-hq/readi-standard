# READI Access — Flow 1: Discover

*Draft 2026.12-pre. How an agent finds where to sign in. Text CC BY 4.0.*

**Goal:** starting from nothing but a resource URL, the agent learns which sign-in server protects it, which permissions exist, and how to identify itself, without any prior setup.

## Steps

**1. The agent calls the resource without a token.**

```http
POST /access/demo/mcp HTTP/1.1
Host: readi.page
```

**2. The resource refuses and points to its metadata** (RFC 9728 §5.1).

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer resource_metadata="https://readi.page/.well-known/oauth-protected-resource/access/demo/mcp", scope="demo.read"
```

**3. The agent reads the protected-resource metadata** (RFC 9728 §3).

```http
GET /.well-known/oauth-protected-resource/access/demo/mcp HTTP/1.1
Host: readi.page
```
```json
{
  "resource": "https://readi.page/access/demo/mcp",
  "authorization_servers": ["https://readi.page/access/demo"],
  "scopes_supported": ["demo.read"],
  "bearer_methods_supported": ["header"],
  "resource_name": "READI Access demo",
  "resource_documentation": "https://readi.page/access/flows/1-discover.md"
}
```

**4. The agent reads the sign-in server's metadata** (RFC 8414 §3, path inserted after `/.well-known/oauth-authorization-server`).

```http
GET /.well-known/oauth-authorization-server/access/demo HTTP/1.1
Host: readi.page
```
```json
{
  "issuer": "https://readi.page/access/demo",
  "authorization_endpoint": "https://readi.page/access/demo/authorize",
  "token_endpoint": "https://readi.page/access/demo/token",
  "response_types_supported": ["code"],
  "grant_types_supported": ["authorization_code"],
  "code_challenge_methods_supported": ["S256"],
  "client_id_metadata_document_supported": true,
  "scopes_supported": ["demo.read"]
}
```

**5. The agent goes to the authorization endpoint**, which starts flow 2 (Introduce) and flow 3 (Ask).

## Requirements

| ID | Requirement | Source |
|---|---|---|
| RA-1.1 | A protected resource MUST answer an unauthenticated request with 401 and a `WWW-Authenticate` header carrying `resource_metadata`. It SHOULD include the `scope` needed. | RFC 9728 §5.1 |
| RA-1.2 | The resource MUST also serve its metadata at the well-known address derived from the resource URL, so agents can find it even if a proxy strips the header. | RFC 9728 §3; MCP SEP-985 |
| RA-1.3 | Each protected endpoint's metadata MUST name that exact endpoint as `resource`. | RFC 9728 §3.3 |
| RA-1.4 | The sign-in server MUST publish RFC 8414 metadata, MUST list `S256` in `code_challenge_methods_supported`, and MUST NOT offer the `plain` method. | RFC 8414; OAuth 2.1 draft (PKCE) |
| RA-1.5 | The sign-in server MUST advertise `client_id_metadata_document_supported: true`. Dynamic client registration is OPTIONAL. | draft-ietf-oauth-client-id-metadata-document (WG draft) |
| RA-1.6 | An agent MUST be able to identify itself with an HTTPS client ID URL (flow 2). An agent that can only use dynamic registration will fail at servers that don't offer it. | Observed (see test record) |

## Test record (November 24, 2026, local build, not published)

Client: the official MCP TypeScript SDK, version 1.32.0.

- **Followed the whole discovery chain:** POST → 401 → protected-resource metadata (200) → sign-in server metadata (200) → authorization page (200).
- **At the authorization page it sent:** an HTTPS client ID URL (a client ID metadata document), the exact `resource` URL, scope `demo.read`, and PKCE with `S256`.
- **Well-known fallback:** with the `WWW-Authenticate` header hidden, it still found the metadata at the well-known address.
- **Without a URL client ID:** the SDK stopped after reading the metadata, because the server doesn't advertise dynamic registration. This is why RA-1.5 and RA-1.6 exist.
- The demo stops before sign-in and token exchange, by design. Each protected endpoint challenged with its own matching metadata.
