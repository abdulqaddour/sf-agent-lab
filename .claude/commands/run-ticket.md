---
description: Run a Notion ticket through build, QA, and (when approved) prod deploy
argument-hint: <ticket ID or Notion link>
---

Run ticket $ARGUMENTS through the sf-agent-lab pipeline. Follow CLAUDE.md.

1. Fetch the ticket from the Notion Tickets database and check its Status.
2. Do the step that matches the Status:
   - **Ready for Pickup** or **In Progress (Build & Test)**: use the **developer** agent to build and
     deploy to test, then the **qa** agent to test. Loop developer → qa until QA Result is Passed,
     or stop after 3 failed QA attempts and report to me.
     When QA passes, the developer opens the PR and moves the card to Ready for Test. Then stop and
     tell me it's ready for my smoke test.
   - **Ready for Test**: do nothing. It's waiting on my smoke test. Tell me that.
   - **Ready for Prod**: re-check that Status is still Ready for Prod, then:
     a. merge the pull request into main (`gh pr merge --merge`), `git checkout main && git pull`
     b. deploy main to prod: `sf project deploy start --target-org prod --source-dir force-app --test-level RunLocalTests`
     c. if it succeeds, set Status to **Done** and Deployed to Prod to today; if it fails, leave the
        status, put the error in QA Notes, and tell me.
   - **To Do/In Draft**, **Done**, or anything else: don't build. Tell me the status and stop.
3. End with a short summary: what changed, the branch and PR, the QA result, and the ticket's new status.
