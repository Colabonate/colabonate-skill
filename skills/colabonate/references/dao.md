# DAO governance

Colabonate's disputes and rules are governed by a DAO. Reads are public; writes need the `dao:write` scope and an explicit instruction from the user for that exact action.

## Reading

- `list_daos` lists all DAOs. The response can be large if image data is inlined; prefer `get_dao`, which returns the foundation DAO.
- `list_proposals` (`daoId?`, `status?`) and `get_proposal` (`proposalId`) show proposals and votes. Statuses: DRAFT, ACTIVE, PASSED, REJECTED, EXECUTED, FAILED.
- `get_governance_config` shows quorum, voting strategies and periods. `get_dao_treasury` shows the balance and recent transactions. `list_dao_members` lists members.
- `daoId` defaults to the foundation DAO, so most calls need no id.

## Writing

- `create_proposal` (`title`, `content`) saves a **DRAFT** by default. `status: "ACTIVE"` publishes it immediately; only do that when the user asked.
- `publish_proposal` opens a draft for voting. Only the author can do this.
- `vote_proposal` with `choice` `FOR`, `AGAINST` or `ABSTAIN`. A vote cannot be taken back lightly: confirm the exact proposal and choice with the user first, and never vote "to be helpful".

## Arbitration

Ticket disputes can go to arbitration. `open_ticket_dispute` starts one; in private-mode tickets the buyer and seller nominate a mutual arbiter. A verdict must be signed by the arbiter with their own key. Never create or sign a verdict on someone else's behalf.
