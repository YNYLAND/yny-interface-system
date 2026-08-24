# ORB DYNAMIC INTERFACE SYSTEM

## Purpose

Dynamic Interface System is the runtime that composes and renders Orb Language using server-side blocks, live state, templates, media, validated actions and channel adapters.

It is distinct from Orb Language System.

- **Orb Language System** defines what Orb can express semantically.
- **Dynamic Interface System** decides how that expression is assembled, validated, rendered and executed in Telegram, Web, Unity, 3D, VR or future channels.

## Core design principle

> Most interface work should be server-side composition, not flagship-model generation.

The system should reuse stored blocks whenever possible:
- images;
- video;
- text fragments;
- entity cards;
- profiles;
- skill/module cards;
- catalogs;
- maps;
- lists;
- checklists;
- state deltas;
- buttons;
- pages;
- modals;
- offers;
- world/space entries.

The flagship model is called only for the semantic slots that actually require reasoning or original language.

## Target pipeline

```text
ACTOR DISCOURSE
↓
COGNITIVE SYSTEM
  understands request / intent / Actor state
↓
GUIDE
  assembles context and scenarios
↓
DYNAMIC INTERFACE COMPOSER
  selects server blocks + templates + actions
↓
OPTIONAL AI SLOT
  scoped prompt to flagship model
↓
ORB LANGUAGE AST
↓
POLICY / RIGHTS / LIVE STATE VALIDATION
↓
CHANNEL RENDERER
  Telegram / Web / Unity / 3D / VR
↓
ACTOR ACCEPT / ACTION
↓
EXECUTOR
↓
STATE DELTA
```

## Server block model

A response can be composed from modules that do not require a flagship call.

Examples:

```text
PROFILE_CARD(profile_id)
ENTITY_CARD(entity_id)
SKILL_CARD(skill_key)
MODE_CARD(mode_key)
IMAGE(asset_id)
VIDEO(asset_id)
KNOWLEDGE_BLOCK(block_id)
CHECKLIST(context_session_id)
STATE_DELTA(delta_id)
MAP(place_set)
WAY(path_id)
TREE(graph_projection)
```

The composer retrieves data from the server and places blocks into the current scenario.

## AI slots

Some blocks are intentionally empty templates with a small scoped prompt.

Example:

```text
AI_REASONING_BLOCK
context:
  selected memory
  current state
  target state
  relevant entity relations
prompt:
  "Briefly explain what these facts mean for the Actor now."
```

The flagship model fills only that block.

This prevents paying the flagship to reproduce data that already exists on the server.

## Guide relationship

Guide works behind the interface as a scenarist/orchestrator.

It may produce a scenario such as:

```text
1. existing PROFILE_CARD
2. existing artifact image
3. current CHECKLIST
4. AI reasoning slot
5. potential WAY block
6. safe navigation button
```

Guide does not need to write the content of blocks that already exist.

## Current Telegram embryo

The live Telegram adapter already demonstrates important pieces of the target architecture:

### Semantic action → renderer
`buildTelegramReplyMarkupFromOrbUi()` maps semantic actions to Telegram inline buttons.

Known renderer cases include:
- `open_cabinet`
- `connect_ynychat` (legacy)
- `enter_ynychat`
- `exit_ynychat`
- `create_profile`.

### Action → executor
`create_profile` callback runs profile creation.

The executor:
1. checks whether a profile link already exists;
2. creates a profile if needed;
3. creates `profile_links` relation to Telegram;
4. returns profile result;
5. renders an open-profile/cabinet button.

### Live wording
The adapter uses `btn.label` when supplied and a fallback otherwise.
The legacy profile flow translates interface text to the user's language.

This is the embryo of a generalized rule:

```text
semantic action = stable
visible wording = contextual where safe
execution = server validated
```

## Current architectural debt

Today UI knowledge is split between:
- `orb-api`;
- `telegram-webhook`;
- channel-specific code;
- skill catalog data;
- model prompt instructions.

For example, Telegram knows how to render `create_profile`, while current `orb-api` v12 does not allow the flagship to emit `create_profile` through its sanitized `[UI]` block.

Target: one canonical action/element registry consumed by all channels.

## Proposed registry fields

Every dynamic interface element/action should eventually have server metadata such as:

```text
element_key
kind
semantic_intent
schema
source_type
renderer_support
label_policy
fallback_label
risk_level
requires_accept
requires_profile
requires_capabilities
requires_connectors
executor_key
reversible
state_delta_type
status
```

## Action safety classes

### Navigation / low risk
May use dynamic contextual wording.

Examples:
- open cabinet;
- open page;
- continue;
- enter an already owned mode.

### State-changing but ordinary
Requires explicit Actor intent or accept.

Examples:
- create profile;
- activate non-financial setting;
- connect approved account.

### Financial / irreversible
Canonical wording and explicit consequences.

Examples:
- buy module;
- transfer funds;
- mint;
- destructive action;
- irreversible state change.

## Core rule

> The flagship supplies intelligence only where intelligence is needed. The server supplies everything it already knows.
