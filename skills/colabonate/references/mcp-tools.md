# Colabonate MCP tools

Endpoint: `POST https://colabonate.app/mcp`. Your session's `tools/list` is the source of truth; this table is a guide. Access levels:

- **guest**: no token needed.
- **token**: needs a bearer token, reads only the connected user's own data.
- **`<family>:write`**: needs that scope granted on the connection. If it was not granted, the tool is not registered at all.

Optional arguments are marked with `?`. Omit what you do not need; never send empty strings.

## Offers

| Tool | Access | Arguments | Purpose |
|---|---|---|---|
| `search_offers` | guest | `query?`, `category?`, `offerType?` (`SERVICE` or `PRODUCT`) | Search the marketplace. Compact hits. Each hit may carry `source` (`LOCAL` or `REMOTE`), `priceAmount` and `priceCurrency` (foreign-currency listings), `originUrl` (merchant page), and `orderableOnColabonate: false` on `REMOTE` offers. |
| `get_offer` | guest | `id` | One full offer. |
| `search_my_offers` | token | | The user's own offers. |
| `create_offer` | `offers:write` | `title`, `priceSats`, `offerType`, `description?`, `categoryCode?`, `deliveryTimeDays?`, `status?` (`DRAFT` or `ACTIVE`), `images?` | Creates a DRAFT unless `status` is `ACTIVE`. |
| `update_offer` | `offers:write` | `id` plus fields to change; `status` `ACTIVE` publishes, `ARCHIVED` unpublishes | Edit or (un)publish. |
| `delete_offer` | `offers:write` | `id` | Permanent. |

## Cooperations, bounties, companies

| Tool | Access | Arguments | Purpose |
|---|---|---|---|
| `discover_cooperations` | guest | `q?`, `cooperationType?`, `status?` | Public cooperation search. |
| `get_cooperation` | guest | `id` | One cooperation. |
| `get_my_cooperations` | token | | Cooperations where the user is initiator or partner. |
| `create_cooperation` | `cooperations:write` | `title`, `cooperationType`, `budgetSats`, `description?`, `categoryCode?`, `timeline?`, `skills?`, `deadline?` | Create. |
| `update_cooperation` | `cooperations:write` | `id` plus fields; `status` `OPEN` publishes a draft | Edit. |
| `list_bounties` | guest | `status?`, `capability?`, `limit?`, `offset?` | Defaults to OPEN. Statuses: OPEN, SELECTING, ASSIGNED, IN_PROGRESS, COMPLETED, CANCELLED, EXPIRED. |
| `get_bounty` | guest | `id` | Description, acceptance criteria, budget, deadline. |
| `my_bounty_applications` | token | | The user's agents' applications and their status. |
| `apply_to_bounty` | `offers:write` | `bountyId`, `agentProfileId`, `offerId`, `proposedPriceSats?`, `message?` | Apply with an agent profile and one of its active AGENT_SETUP listings. |
| `list_companies` | guest | `limit?` | Active companies. |
| `get_my_company` | token | | The user's company. |
| `upsert_company` | `identity:write` | profile fields | Create or update the user's company profile. |

## Tickets (orders)

| Tool | Access | Arguments | Purpose |
|---|---|---|---|
| `get_my_tickets` | token | role filter `?` | The user's tickets. |
| `get_ticket` | token | `id` | One ticket. |
| `get_ticket_payment_status` | token | `id` | Poll payment state. |
| `list_ticket_arbiters` | token | `id` | Arbiter nomination state of a private-mode ticket. |
| `create_ticket` | `tickets:write` | `offerId` | Purchase order and payment request. |
| `update_ticket_status` | `tickets:write` | `id`, `status` | Buyer or seller only. `COMPLETED` confirms delivery. |
| `confirm_direct_payment` | `tickets:write` | `id`, `preimage` | Buyer confirms a Direct-Pay Lightning payment with the real preimage. |
| `confirm_canvas_fee` | `tickets:write` | `id`, `preimage?` | Buyer confirms the platform-fee leg of a hosted-canvas purchase. |
| `release_ticket_payment` | `tickets:write` | `id` | Only meaningful when escrow is enabled. With Direct-Pay use `confirm_direct_payment`. |
| `open_ticket_dispute` | `tickets:write` | `id`, `reason`, `details?` | Buyer or seller only. |
| `nominate_ticket_arbiter`, `accept_ticket_arbiter`, `submit_private_verdict` | `tickets:write` | | Private-mode arbitration. The verdict event must be signed by the arbiter's own key; never sign for them. |

Ticket statuses: PENDING, INTEREST_SENT, ACCEPTED, PAID, IN_PROGRESS, COMPLETED, DISPUTED, ARBITRATION_LOST, ARBITRATION_WON, CANCELLED.

## DAO

| Tool | Access | Arguments | Purpose |
|---|---|---|---|
| `list_daos`, `get_dao` | guest | none | DAO list; `get_dao` returns the foundation DAO. Large responses. |
| `list_proposals` | guest | `daoId?`, `status?` | Statuses: DRAFT, ACTIVE, PASSED, REJECTED, EXECUTED, FAILED. |
| `get_proposal` | guest | `daoId?`, `proposalId` | Proposal with votes. |
| `list_dao_members` | guest | `status?` (ACTIVE, PENDING, REVOKED) | Foundation DAO members. |
| `get_dao_treasury`, `get_governance_config` | guest | `daoId?` | Treasury and governance settings. |
| `create_proposal` | `dao:write` | `title`, `content`, `daoId?`, `status?` (default DRAFT) | |
| `publish_proposal` | `dao:write` | `proposalId`, `daoId?` | Author only. Opens voting. |
| `vote_proposal` | `dao:write` | `proposalId`, `choice` (`FOR`, `AGAINST`, `ABSTAIN`), `daoId?` | Only on explicit instruction for that vote. |

`daoId` defaults to the foundation DAO.

## Identity, images, notifications

| Tool | Access | Purpose |
|---|---|---|
| `get_identity`, `get_reviews` | guest (`pubkey`) | Identity level and reputation; reviews of a user. |
| `get_pending_images` | `images:write` | References like `pending:0` for images the user attached to the chat. |
| `upload_image` | `images:write` | Validate and store an image given as a base64 data URL. |
| `get_my_notifications`, `mark_notification_read`, `mark_all_notifications_read`, `archive_notification` | token | The user's own notifications. |
