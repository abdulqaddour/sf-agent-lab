# sf-agent-lab: shared rules for every agent

This is a Salesforce DX project for a learning lab: an agentic DevOps pipeline.
A developer agent and a QA agent build and test Salesforce changes in a test org,
and changes reach prod only after two human checkpoints by Abdulrahman.

## Orgs (Salesforce CLI aliases)
- `test`: where all building and testing happens. It is the default target org.
- `prod`: production. Only deployed to from `main`, and only for tickets in **Ready for Prod**.
- Always pass the org explicitly, and put it right after the command,
  e.g. `sf project deploy start --target-org test --source-dir force-app`.
  The permission rules in `.claude/settings.json` rely on this order.
- Never run `sf org delete`, `sf org logout`, or anything that changes org auth.

## Ticket board (Notion)
- Board page: https://app.notion.com/p/4173f0939ee8817aacc7fb51741f29a1
- Tickets database: https://app.notion.com/p/7e07e6a1a1d34120a641e8bda638dffa
  (data source `collection://1ffab743-8d19-413e-b6bb-2bc6dfcdd2c5`)
- Ticket fields: Ticket ID, Status, Type, Priority, User Story/Context,
  Steps to Reproduce (if bug), Acceptance Criteria, Metadata Touched,
  Git Branch, Pull Request, QA Result, QA Attempts, QA Notes, Deployed to Prod.

### Statuses and who moves cards
| Status | Meaning | Who moves a card INTO it |
|---|---|---|
| To Do/In Draft | Ticket drafted, not reviewed | Anyone (agents may draft tickets here) |
| Ready for Pickup | **Checkpoint 1**: Abdulrahman approved it | **Abdulrahman only** |
| In Progress (Build & Test) | Developer builds, QA tests, loop until QA passes | Developer agent |
| Ready for Test | Built and QA-passed; waiting on Abdulrahman's smoke test | Developer agent, only after QA Result = Passed |
| Ready for Prod | **Checkpoint 2**: Abdulrahman smoke-tested and approved | **Abdulrahman only** |
| Done | Merged to main and deployed to prod | Agent, after a successful prod deploy |

Hard rules:
- Never move a card into **Ready for Pickup** or **Ready for Prod**. Those are human checkpoints.
- Never start work on a ticket that is not in **Ready for Pickup**.
- Never deploy to `prod` unless the ticket is in **Ready for Prod** right now (re-check Notion just before deploying).
- If a smoke test fails, Abdulrahman moves the card back to In Progress (Build & Test) with notes; treat that as new work.
- Work one ticket at a time: the single test org can only hold one ticket's changes cleanly.

## Git
- One repo, `main` = what is in prod. Never commit directly to `main`.
- Branch per ticket: `SFLAB-<id>-short-description` (e.g. `SFLAB-3-renewal-date-validation`).
- Commit messages start with the ticket ID: `SFLAB-3: Add validation rule for Renewal Date`.
- Open a pull request with `gh pr create` when the ticket moves to Ready for Test; put the PR link on the ticket.
- Merge to `main` only after the ticket is in Ready for Prod.

## Salesforce conventions
- Metadata lives in `force-app/main/default/`.
- Every new field ships with a permission set update so users can actually see it
  (use or create the `SF_Agent_Lab` permission set).
- Apex needs test classes with at least 75% coverage; run tests in `test`.
- If something was changed by clicking in an org, retrieve it before editing files so git stays in sync.

## Safety
- Read freely. Ask Abdulrahman before anything destructive or irreversible
  (deleting metadata, destructive changes, data deletes).
- If the ticket is unclear or contradicts itself, stop and leave a question in the ticket's QA Notes
  instead of guessing.
