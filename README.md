# CrowdFlex — Flexible Crowdfunding Contract

A flexible, controlled crowdfunding Clarity contract for Stacks. CrowdFlex supports campaign creation, contributions, manual refunds, fund claims by creators, admin controls, and read-only views.

## Features

- Create campaigns with goal and duration.
- Contribute STX to active campaigns.
- Creators can claim funds if goal met.
- Contributors can request refunds if goal not met after deadline.
- Campaign pause/resume, extend deadline, update goal (only if no funds raised).
- Admin operations: remove campaign, force-cancel (mark inactive), transfer admin.
- Read-only views for campaign and contribution state.
- Explicit error codes for common failure modes.

## Contract interface

Public functions
- create-campaign (goal uint) (duration uint) -> (ok id)
- contribute (id uint) (amount uint) -> (ok true)
- claim-funds (id uint) -> (ok true) / transfers funds to creator
- get-refund (id uint) -> (ok true) / refunds contributor
- cancel-campaign (id uint) -> (ok true)
- extend-deadline (id uint) (extra uint) -> (ok true)
- update-goal (id uint) (new-goal uint) -> (ok true)
- admin-remove (id uint) -> (ok true)
- admin-force-cancel (id uint) -> (ok true)  (marks inactive; contributors still call get-refund)
- toggle-campaign-active (id uint) -> (ok new-state)
- transfer-admin (new-admin principal) -> (ok true)

Read-only views
- get-campaign (id uint)
- get-contribution (id uint) (user principal)
- get-campaign-count
- is-active (id uint)

## Data layout

- next-campaign-id: uint
- contract-admin: principal
- map campaigns: campaign-id -> { creator, goal, deadline, raised, claimed, active }
- map contributions: { campaign-id, user } -> uint

## Error codes

- ERR_DEADLINE_PASSED (u100)
- ERR_NOT_FOUND (u101)
- ERR_UNAUTHORIZED (u102)
- ERR_GOAL_NOT_MET (u103)
- ERR_ALREADY_CLAIMED (u104)
- ERR_NOTHING_TO_REFUND (u105)
- ERR_INACTIVE (u106)
- ERR_ALREADY_INACTIVE (u107)
- ERR_CANNOT_UPDATE_GOAL (u108)

## Quickstart (local development)

1. Install Clarinet: https://github.com/hirosystems/clarinet
2. From the repository root:
   - Run local node and tests:
     ```bash
     clarinet test
     ```
   - Launch REPL / console:
     ```bash
     clarinet console
     ```
3. In Clarinet console or test scripts, example calls:

   Create campaign (goal 1000 STX, duration 100 blocks):
   ```clojure
   (contract-call? .flexifund create-campaign u1000 u100)
   ```

   Contribute 50 STX to campaign id 0:
   ```clojure
   (contract-call? .flexifund contribute u0 u50)
   ```

   Claim funds (creator of campaign 0):
   ```clojure
   (contract-call? .flexifund claim-funds u0)
   ```

   Request refund (contributor, after deadline and if goal not met):
   ```clojure
   (contract-call? .flexifund get-refund u0)
   ```

   Toggle campaign active state (creator):
   ```clojure
   (contract-call? .flexifund toggle-campaign-active u0)
   ```

   Transfer admin (current admin):
   ```clojure
   (contract-call? .flexifund transfer-admin 'SP2...NEWADMIN)
   ```

## Testing

- Write Clarinet tests in tests/ to simulate flows: create, contribute, deadline expiry, claim, refund, admin actions.
- Use assertions to verify storage maps, transfers, and error responses.

## Security notes & limitations

- Refunds are manual: contributors must call get-refund to receive refunded STX.
- admin-force-cancel only marks a campaign inactive; it does not perform refunds automatically.
- update-goal only allowed if raised == 0 to prevent post-contribution goal manipulation.
- Consider adding batched refund helpers or automated refund flows for UX improvement.
- Validate transfer amounts and caller contexts in front-end integrations.
epository root.

## Contact

Refer to repository issues and PRs for questions or feature requests.
