# READI specification

*Draft 2026.11, open for comment. READI is an open, testable standard for how ready a business website is for AI agents. Licensed under CC BY 4.0. Published at https://readi.page/spec.*

## Readiness levels

Levels are cumulative. A site reaches a level by meeting its required checks and the required checks for every lower level.

| Level | Name | Meaning |
|---|---|---|
| R0 | Reachable | The site loads over HTTPS, with no bot walls for permitted agents. |
| R1 | Findable | Search and AI crawlers are allowed, and the key business facts are in the page itself. |
| R2 | Readable | Structured data (for example Schema.org LocalBusiness JSON-LD) matches what the page shows: name, hours, services, prices, location. |
| R3 | Usable | Real contact and quote paths an agent can find and use (phone, email, form, booking link). |
| R4 | Actionable | Agent endpoints that validate (for example an MCP server or an agent card) so an agent can send an inquiry or request. |
| R5 | Transactable | Booking or payment an agent can complete on the person's behalf. |

### R0 Reachable

The site loads over HTTPS, with no bot walls for permitted agents.

**What’s checked**

- The home page and key public pages load over HTTPS with a valid certificate.
- An allowed agent can retrieve the same public information people receive, without a login or interactive challenge.

**How to get started**

1. Serve the public site over HTTPS and redirect HTTP to HTTPS.
2. Test the public pages from outside your own network, including the behavior for permitted agents.

### R1 Findable

Search and AI crawlers are allowed, and the key business facts are in the page itself.

**What’s checked**

- Search crawlers and permitted assistant fetchers can access the key public pages.
- The public pages are linked and key facts are present in the initial HTML.

**How to get here from R0**

1. Check robots rules against the search and assistant access you intend to permit.
2. Give important facts a stable public URL and put them in the initial HTML.

### R2 Readable

Structured data (for example Schema.org LocalBusiness JSON-LD) matches what the page shows: name, hours, services, prices, location.

**What’s checked**

- Structured data parses and describes the real business, service, or resource using an appropriate type.
- The structured facts agree with visible content, including hours, services, prices, and locations when they apply.

**How to get here from R1**

1. Add structured data for the facts already published on the page.
2. Validate the data and compare each published field with the visible facts. Update both when a fact changes.

### R3 Usable

Real contact and quote paths an agent can find and use (phone, email, form, booking link).

**What’s checked**

- A public page links to at least one relevant, working contact or quote path.
- An agent can identify the destination and what information is needed before a request is sent.

**How to get here from R2**

1. Publish descriptive links to the ways people can actually contact you or request a quote.
2. Check the path yourself and label any required fields or next steps.

### R4 Actionable

Agent endpoints that validate (for example an MCP server or an agent card) so an agent can send an inquiry or request.

**What’s checked**

- An agent can discover a documented endpoint and validate its format and applicable protocol.
- A permitted agent can send a test inquiry or request and receive a clear outcome.

**How to get here from R3**

1. Publish a discoverable endpoint with a documented request and response schema.
2. Test authorization, errors, and confirmations in a safe test environment.

The [READI Access working draft](../access/README.md) maps R4 to flows 1–3 and 5 for at least one documented action, plus flow 7 for revocation.

### R5 Transactable

Booking or payment an agent can complete on the person's behalf.

**What’s checked**

- An authorized agent can complete a supported booking or payment on a person’s behalf.
- The flow presents the material terms before confirmation and returns an outcome or receipt afterward.

**How to get here from R4**

1. Document the supported booking or payment flow and where approval is required.
2. Test a full authorized transaction in a sandbox, including success, failure, and duplicate submissions.

The [READI Access working draft](../access/README.md) maps R5 to the R4 flows, plus flow 6 for every payment or booking and flow 4 where the person may be away.

## Check states

Every check returns exactly one state.

| State | Meaning |
|---|---|
| Found and valid | The check found a feature and the evidence satisfies the test. |
| Found but invalid | The feature exists and the evidence identifies a specific problem. |
| Absent | The check did not find the feature. |
| Unsupported | The checker does not support the mechanism needed to evaluate the feature. |
| Not applicable | The check does not apply to the site or the claim being tested. |
| Unverified | The available evidence is insufficient to decide the check. |

Absence and invalidity are distinct outcomes. Only “Found and valid” counts as a pass; “Unverified” and “Unsupported” are never shown as passes. Optional extras earn badges and never count as deductions.

## Evidence

Every check records a state and the evidence behind it.

- Check identifier, tested level, and spec version.
- Target URL, evaluation time in UTC, and crawler purpose when relevant.
- Test method, expected behavior, and observed behavior.
- Check state, explanation, and a reproducible reference such as an HTTP status, relevant excerpt, or validation result.
- Any access restrictions or test limitations that affect the result.

## Access by purpose

Record search, assistant access, and AI training permissions separately. Blocking AI training never costs search or assistant points.

Report how readily agents can find a site separately from what they can do with it. Finding a page provides evidence for discovery. Exercising a working path provides evidence for usability.

## Principles

- **Evidence for every check.** Each check records a state and the evidence behind it. A feature can be absent, or present and broken. The report distinguishes the two.
- **Access by purpose.** Search, assistant access, and AI training are different purposes. Blocking AI training never costs search or assistant points.
- **Two readiness measures.** We report how readily agents can find a site separately from what they can do with it.
- **Optional badges.** Optional extras can earn badges. They never lower a readiness level.
- **A draft crosswalk.** A READI report is designed to show corresponding levels or checks in other frameworks. Mappings stay draft until published.
- **Open governance.** The standard is published under CC BY 4.0. Releases are dated, with tests published alongside the spec.

## Draft crosswalk

The crosswalk will show corresponding levels or checks in other frameworks: OARS levels, AgentHermes ARL readiness levels, and the audit categories of Cloudflare’s Agent Readiness scanner. Numerical mappings are draft and have yet to be published.

## Versioning and governance

READI uses date-based versions. This first draft is 2026.11. A report identifies the spec version it used so its result can be reproduced.

The standard is open under CC BY 4.0 for anyone to use. Implementation tests are published alongside each spec version. This draft describes the intended governance.

Comment on the draft by [opening an issue](https://github.com/yolo-labs-hq/readi-standard/issues/new/choose).
