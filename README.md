# READI

READI is an open, testable standard for how ready a business website is for AI agents: whether agents can reach it, find it, read it, use it, act on it, and complete a transaction on a person’s behalf. Human-readable versions are at [readi.page](https://readi.page). READI is sponsored by [The Radar Network](https://theradarnetwork.com/).

## Documents

| Document | Version | Status |
|---|---|---|
| [READI specification](spec/README.md) | 2026.11 | Draft, open for comment |
| [READI Access](access/README.md) | 2026.12-pre | Working draft, not for implementation |
| [READI Access flows 1–7](access/flows/) | 2026.12-pre | Working draft, not for implementation |

## The six levels

| Level | Name | In short |
|---|---|---|
| R0 | Reachable | Loads over HTTPS, with no bot walls for permitted agents |
| R1 | Findable | Crawlers allowed, and key facts in the page itself |
| R2 | Readable | Structured data that matches what the page shows |
| R3 | Usable | Real contact and quote paths an agent can use |
| R4 | Actionable | Agent endpoints that validate and accept a request |
| R5 | Transactable | Booking or payment an agent can complete for a person |

Levels are cumulative. See the [specification](spec/README.md) for the checks behind each one.

## Comment on the drafts

Comments are welcome from site owners, agent builders, standards people and anyone else affected. [Open an issue](https://github.com/yolo-labs-hq/readi-standard/issues/new/choose) and pick the part of the draft you’re commenting on. See [how to comment](CONTRIBUTING.md) for what helps most.

## License

The text in this repository is licensed under [Creative Commons Attribution 4.0 International](LICENSE) (CC BY 4.0).
