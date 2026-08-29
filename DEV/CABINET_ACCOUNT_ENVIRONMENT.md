# Cabinet Account DEV Environment

This branch is an isolated development environment for the PROFILE → ACCOUNT vertical slice.

## Isolation rules

- Do not deploy this branch to the production Cabinet site.
- Production frontend (`cabinet.yny.land`) must remain unchanged until DEV acceptance.
- Account data must use the separate Supabase project `YNY DEV` only.
- Test purchases use test YNY balances in DEV and must not write to production balances or transaction tables.
- Any Cloudflare deployment must use a separate DEV Pages/Worker project and/or DEV subdomain; never the production Cabinet project.
- Promotion to production happens only after explicit review and approval.

## Current DEV branch

`cabinet-account-dev`

## Current scope

- Account list
- Repeatable Account purchase: 1 YNY = one blank Account container
- Account setup (name/avatar)
- Account workspace
- Installable modules (PAGE, GROUP, etc.)
- Manual list ordering

ACCOUNT remains a neutral container. PAGE/GROUP are modules installed inside it, not Account forms.
