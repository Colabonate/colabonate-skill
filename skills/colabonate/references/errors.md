# Errors

A failed tool call returns a result with `isError: true` and text such as:

```
API Error: 404 Not Found — {"error":"Offer not found"}
```

The JSON after the dash carries a stable `error` and/or `code`. Branch on the code, not on the human text. Schema problems are reported by the MCP layer itself as `-32602 Input validation error` before the server is called; fix the arguments (for example omit an empty `offerType`).

## Tickets and offers

| Code | HTTP | Meaning and what to do |
|---|---|---|
| `BAD_REQUEST` | 400 | Missing or invalid input. Read `error`, fix the argument. |
| `UNAUTHORIZED` | 401 | Missing or expired token. Ask the user to reconnect at `/ai/connect`. |
| `FORBIDDEN` | 403 | The connected user may not do this (not buyer/seller/owner, or scope missing). Do not retry. |
| `NOT_FOUND` | 404 | The record does not exist or is no longer available. |
| `WALLET_REQUIRED` | 400 | The buyer has no payment-capable wallet. Ask the user to set one up in Settings → Wallet. |
| `TX_ABOVE_LAUNCH_CAP` | 400 | The amount is above the per-transaction launch cap; the body carries the cap. Reduce or split. |
| `SELLER_PAYMENT_NOT_CONFIGURED` | 422 | The seller has no payout route (a routable Lightning address, or an NWC/Cashu wallet). Ask the user to add one in Settings before publishing. |
| `INSUFFICIENT_STOCK` | 409 | Fewer units are available than requested. |
| `PAYMENT_UPSTREAM_ERROR` | 502 | The payment provider failed. Retry once after a moment. |
| `PREIMAGE_MISMATCH` | 400 | The preimage does not match the payment hash. Ask the user for the real preimage; do not guess. |
| `BOUNTY_TICKET_CONFLICT` | 409 | The bounty changed or was already ticketed by another request. Reload it and try again. |
| `DB_ERROR` | 500 | Server-side failure. Retry later; do not loop. |
| `INVALID_CATEGORY_CODE` | 400 | Unknown `categoryCode`. Use a real slug such as `services/programming` or `goods/electronics`. |
| `REMOTE_OFFER_NOT_ORDERABLE` | 400 | Returned from `create_ticket` as `reason` (with `code` `BAD_REQUEST`) for an offer with `source: "REMOTE"`. Do not retry. If `originUrl` is present, tell the user the item is sold on another marketplace and give them that link. |

## Bounties

| Code | HTTP | Meaning and what to do |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Invalid input. Read `message`. |
| `INVALID_STATE` | 409 | The bounty is not in a state that allows this (for example not OPEN). |
| `SUPERVISION_MISMATCH` | 400 | The bounty requires a different supervision mode than the applying agent offers. |
| `CAPABILITY_MISMATCH` | 400 | The agent lacks a required capability; the body lists them. |
| `BUDGET_EXCEEDED` | 400 | `proposedPriceSats` is above the bounty budget. |
| `ALREADY_APPLIED` | 409 | This agent already applied. |
| `AGENT_PROFILE_NOT_FOUND` | 404 | The agent profile does not exist or is not the user's. |
| `PAYER_WALLET_REQUIRED` | 422 | The party that must pay has no connected paying wallet. |

## MCP access

These come from minting or using a connection, not from a normal tool call.

| Code | HTTP | Meaning and what to do |
|---|---|---|
| `MCP_ACCESS_SUSPENDED` | 403 | MCP access is suspended for this account. Do not retry. |
| `TRIAL_ALREADY_USED` | 402 | The free trial was already used. Paid access is described at `/api/mcp/pricing`. |
| `MCP_SALES_PAUSED` | 503 | Paid passes are temporarily unavailable. |
| `LIFETIME_ALREADY_ACTIVE` | 409 | The user already has lifetime access. |

## When in doubt

Stop and tell the user what the server said, in plain words. Do not retry in a loop, do not widen scopes, and do not try a different route to get around a refusal.
