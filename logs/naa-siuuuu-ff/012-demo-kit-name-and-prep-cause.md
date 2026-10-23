# Demo Kit Name and Prep Cause

## Goal
Fix customers seeing demo-kit café names with a hash suffix, and use the user's answers to narrow down why Prep Advisor falls back on dev.

## Scope
One user-assistant exchange with two related tasks, both in demo-kit workspaces.

## Key prompts
- "From development. soclaas ... Yes it does (brew finder shows Ai suggestion), and refresh advice does not appear to work." This answered the diagnosis questions: the dev environment, the SoCLaaS provider, a working Brew Finder, and a refresh that always fails.
- "A bug I need you to fix, the customer appears to see the cafe name with some hash". This came with screenshots showing "Synthetic Coffee 7bd8704098e6625948f5192228e8ba83".

## Decisions and corrections
- The hash is the 32-hex kit code that `publish_demo_kit` (SYS-12, owned by shawnnygoh) appended to both kit café names and to order café-name snapshots. SYS-12 does not name the café, there is no name-uniqueness constraint, and kits are isolated workspaces, so the suffix was unnecessary.
- The applied migration was not edited. A new migration, `20261023000600_demo_kit_cafe_names.sql`, redefines the function with only those four literals changed; a diff against the original confirmed that.
- Existing kits and their order snapshots were left unchanged, because purchase snapshots are preserved and kits are never touched after creation. Users need a new kit to see the fix.
- The migration was applied to the shared local database with `create or replace` (non-destructive); no reset.
- Checks: demo-kit integration tests (54) passed and produced kits named "Synthetic Coffee". `bun run verify` passed (1410 tests), and the demo-kit e2e tests (3) passed. The commit was pushed on `fix/demo-kit-cafe-name`, and PR #189 asks shawnnygoh to review.
- Prep Advisor: since Brew Finder works with the same provider, the dev settings and key are fine. Local kit data shows each item's weekday quantities are 7, 7, 9, 8, 7, 9, 8, a total of 55. OWN-07.16 allows "more" only on 8 or 9 and "less" only on 7, and one invalid suggestion rejects the whole result. The spec-fixed prompt never states this rule, so near-flat data makes a rejected answer likely. The fix needs a spec decision because INTEGRATION-SPEC fixes the prompt text verbatim.

## Files created or modified
- `supabase/migrations/20261023000600_demo_kit_cafe_names.sql` (branch `fix/demo-kit-cafe-name`, PR #189)
