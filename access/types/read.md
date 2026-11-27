# READI Access type: Read

*Draft 2026.12-pre, proposed. Part of READI Access (https://readi.page/access). Text CC BY 4.0.*

`https://readi.page/access/types/read`

**See information at one site. Can’t make, change or cancel anything.**

Use this URL as the `type` of an `authorization_details` entry (RFC 9396). Send `Accept: application/json` for the same definition as JSON.

## Actions

| Action | Shown to the person as |
|---|---|
| `read` | See information |

## Fields

| Field | Required | Format | Meaning |
|---|---|---|---|
| `type` | Yes | URI | `https://readi.page/access/types/read` |
| `locations` | Yes | array of URIs | The resource the permission is for (RFC 9396 §2.2), equal to the token’s `resource` (RA-3.7). Exactly one entry. |
| `actions` | Yes | array of strings | Exactly `["read"]`. |
| `datatypes` | No | array of strings | What can be seen, as the resource names it, for example `["bookings"]` (RFC 9396 §2.2). Without it, everything the resource offers to read. |

## On the approval screen

With datatypes:

> Let BookingBot see your bookings at Café Example? It can’t make, change or cancel anything. Access ends in 10 minutes.

## Rules

- A read permission never authorizes a change of state. The AS rejects a read entry with any action other than `read` (`invalid_authorization_details`, RA-3.4).
- It’s its own type, not a `read` action on another type, so permission to look is never confused with permission to act (RA-3.12).
- It creates no obligation, so a strict standing approval may cover it (§9.1).
- The approval screen, the person’s review list (flow 7) and introspection show it as its own permission.

## Example

```json
[
  {
    "type": "https://readi.page/access/types/read",
    "actions": [
      "read"
    ],
    "locations": [
      "https://cafe.example/api/bookings"
    ],
    "datatypes": [
      "bookings"
    ]
  }
]
```
