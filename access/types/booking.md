# READI Access type: Booking

*Draft 2026.12-pre, proposed. Part of READI Access (https://readi.page/access). Text CC BY 4.0.*

`https://readi.page/access/types/booking`

**Make, change or cancel a booking at one place, on the terms shown.**

Use this URL as the `type` of an `authorization_details` entry (RFC 9396). Send `Accept: application/json` for the same definition as JSON.

## Actions

| Action | Shown to the person as |
|---|---|
| `create` | Make a booking |
| `change` | Change an existing booking |
| `cancel` | Cancel an existing booking |

## Fields

| Field | Required | Format | Meaning |
|---|---|---|---|
| `type` | Yes | URI | `https://readi.page/access/types/booking` |
| `locations` | Yes | array of URIs | The resource the permission is for (RFC 9396 §2.2), equal to the token’s `resource` (RA-3.7). Exactly one entry. |
| `actions` | Yes | array of strings | One or more of `create`, `change`, `cancel`. |
| `start` | No | RFC 3339 date-time with `Z` or a numeric offset | The exact start. Required for an exact booking (flow 6). One instant, never a bare local time (RA-6.9). |
| `zone` | No | IANA time zone name | The venue’s zone, for showing the local time (RFC 9557). Doesn’t change the instant in `start`. |
| `party_size` | No | integer ≥ 1 | How many people the booking is for. |
| `item` | No | string | What’s being booked, as the person should see it, for example “Table for 4”. |
| `price` | No | { amount, currency } | Price, as a decimal string and an ISO 4217 code. |
| `deposit` | No | { amount, currency, refundable_until } | Deposit, and until when it’s refundable (RFC 3339). |
| `cancellation` | No | string | The cancellation terms in plain language. |
| `reference` | No | string | For `change` and `cancel`: the booking’s reference at the resource. |
| `transaction_id` | No | string | The transaction this approval is for (flow 6). Execution is idempotent for it (RA-6.8). |
| `max_uses` | No | integer ≥ 1 | How many times the permission can be used. Defaults to 1. An exact booking is always 1. |
| `max_party_size` | No | integer ≥ 1 | Limit form: the largest party allowed, instead of an exact `party_size`. |
| `before` | No | RFC 3339 date or date-time | Limit form: the latest start allowed, instead of an exact `start`. |

## On the approval screen

Exact (flow 6):

> Book a table for 2 at Café Example on Saturday, November 28, 2026, at 7:00 pm (America/Denver, UTC−7), once. Free cancellation until 24 hours before.

Limits:

> Book tables at Café Example for up to 4 people, starting before December 5, 2026.

## Rules

- A booking creates an obligation for the person, so each one needs a fresh, exact approval (flow 6) unless a bounded standing approval covers it (§9). A strict standing approval never covers it (RA-9.3).
- For `create`, an exact request carries `start` and `party_size` (or `item`). The limit form (`max_party_size`, `before`) is for asking ahead or for standing approvals; the resource still checks each booking against it.
- The approval screen shows the date, local time and zone or offset, every price, deposit and cancellation term, and how many times it can be used.
- The resource enforces every field it receives. A request outside them is refused, not trimmed.

## Example

```json
[
  {
    "type": "https://readi.page/access/types/booking",
    "actions": [
      "create"
    ],
    "locations": [
      "https://cafe.example/api/bookings"
    ],
    "start": "2026-11-28T19:00:00-07:00",
    "zone": "America/Denver",
    "party_size": 2,
    "max_uses": 1
  }
]
```
