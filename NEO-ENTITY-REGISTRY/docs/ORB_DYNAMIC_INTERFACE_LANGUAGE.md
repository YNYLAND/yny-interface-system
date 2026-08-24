# ORB DYNAMIC INTERFACE LANGUAGE — SPLIT NOTE

This document name previously mixed two separate systems.

The architecture is now split into:

1. **ORB LANGUAGE SYSTEM** — the channel-independent semantic language Orb speaks: TEXT, CARD, BUTTON, ACTION, CHECK, STATE_DELTA, OFFER, ACCEPT, AVATAR, WORLD, etc.
   - Canonical document: `YNYLAND/yny-orb-core/docs/ORB_LANGUAGE_SYSTEM.md`

2. **ORB DYNAMIC INTERFACE SYSTEM** — the server-side composer/runtime that selects stored blocks, validates state/actions and renders Orb Language into Telegram, Web, Unity, 3D, VR and future channels.
   - Canonical document: `NEO-ENTITY-REGISTRY/docs/ORB_DYNAMIC_INTERFACE_SYSTEM.md`

Canonical distinction:

```text
ORB LANGUAGE SYSTEM
= what Orb means / expresses

ORB DYNAMIC INTERFACE SYSTEM
= how the system assembles, renders and executes that expression
```

The previous combined design is now represented by these two canonical specifications.
