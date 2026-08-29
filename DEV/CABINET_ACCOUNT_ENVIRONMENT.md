# Cabinet Account DEV Environment

The accessible GitHub repositories/branches are development and test space only. They are not the source of the live production Cabinet deployment.

## Isolation rules

- GitHub work is DEV/test by default.
- Supabase Account data must use the separate project `YNY DEV` only.
- Test purchases use test YNY balances in DEV and must not write to production balances or production transaction data.
- Any Cloudflare deployment for this work must use a separate DEV Pages/Worker project and/or DEV hostname.
- The live production Cabinet is a separate published environment and must not be modified until explicit acceptance.

## Current feature branch

`cabinet-account-dev`

This extra branch is only for feature isolation/convenience inside the already-test GitHub environment; it is not a production boundary.

## Current scope

- Account list
- Repeatable Account purchase: 1 YNY = one blank Account container
- Account setup (name/avatar)
- Account workspace
- Installable modules (PAGE, GROUP, etc.)
- Manual list ordering

ACCOUNT remains a neutral container. PAGE/GROUP are modules installed inside it, not Account forms.
