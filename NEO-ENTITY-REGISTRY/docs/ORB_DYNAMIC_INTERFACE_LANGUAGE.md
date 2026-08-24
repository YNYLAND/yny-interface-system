# ORB DYNAMIC INTERFACE LANGUAGE

## Purpose

Dynamic Interface is the visual/action language of Orb.

The flagship model should express semantic interface intent. The server validates intent against current state, rights, prices and available actions. Channel adapters render the same semantic action in Telegram, Web, Unity, 3D, VR or future interfaces.

```text
FLAGSHIP ORB
→ RESPONSE + UI INTENTS
→ POLICY / RIGHTS / LIVE STATE VALIDATOR
→ CANONICAL UI AST
→ CHANNEL ADAPTER
→ Telegram / Web / Unity / 3D / VR / future interface
```

## Current production reality

Today a real proto-language already exists, but it is fragmented across four layers.

### 1. Flagship/model UI layer

Current `orb-api` allows the flagship response to emit a hidden `[UI]...[/UI]` block.

Model-originated actions accepted by current `sanitizeOrbUi` are:
- `open_skill_catalog`
- `open_skill` + `skill_key`
- `enter_ynychat` when owned and not already active
- `exit_ynychat` while YNY CHAT is active.

The flagship can suggest semantic UI, but the server sanitizes it before rendering.

### 2. Server-generated UI layer

`orb-api` also creates UI independently of the model.

Current server UI structures include:
- `buttons`
- `skill_cards`
- `modes`
- `catalog_mode_key`
- `list`
- state data.

Server actions currently include:
- `get_list_state`
- `purchase_list_slots`
- `get_mode_state`
- `open_skill_catalog`
- `toggle_mode`
- `open_skill`
- `purchase_skill`
- `enter_ynychat`
- `exit_ynychat`.

`purchase_skill` is intentionally server-generated from a validated skill card; the model cannot invent a BUY action, price or dependency bundle.

### 3. Telegram rendering/action layer

The current Telegram adapter explicitly understands a broader visible action vocabulary:
- `open_cabinet`
- legacy `connect_ynychat`
- `enter_ynychat`
- `exit_ynychat`
- `create_profile`
- Orb Web continuation / handoff.

Therefore the **observable Telegram language is broader than the current orb-api model whitelist**.

This is important: when auditing what Orb can visibly say with buttons, we must inspect the complete channel path, not only `sanitizeOrbUi`.

### 4. Dynamic label / localization layer

Current Telegram behavior already separates action semantics from some visible wording.

The adapter uses an incoming `btn.label` when one is supplied and falls back to a default label when it is absent.

Profile creation flow also calls GPT-based `translateText(...)` for user-facing phrases including the button meaning `Open profile`. Because this is model-generated translation rather than a fixed locale dictionary, visible wording may vary from one execution/language/context to another.

So a live button effectively already has two layers:

```text
STABLE ACTION
create_profile / open_cabinet / enter_ynychat / ...

+

DYNAMIC SURFACE LABEL
"Создать профиль"
"Открыть профиль"
"Перейти в кабинет"
... wording/localization may vary
```

This variability should not be treated as an error by default. It can become a deliberate feature of the Dynamic Interface Language: **stable semantic action + context-sensitive natural-language surface**.

For actions where exact legal/economic wording matters, the server may force a canonical label.

## Profile creation today

A production Telegram-specific profile mechanism definitely exists.

Current direct phrase path:

```text
Telegram message contains “создай профиль”
→ telegram-webhook intercepts before flagship Orb
→ checks profile_links
→ if profile exists: reads profile
→ otherwise creates next profile number
→ inserts profiles row
→ inserts Telegram profile_link
→ sends result
→ renders Open profile/cabinet button
```

The Telegram adapter also contains a `create_profile` callback renderer/handler vocabulary, so `CREATE PROFILE` is already part of the channel's practical interface language even though it is not currently part of the `orb-api` model whitelist.

A separate legacy `orb-create-profile` Edge Function duplicates much of the creation logic.

Target architecture is not to remove this working behavior, but to **lift it into one canonical action**:

```text
Actor intent
→ ACTION / PROFILE_CREATE
→ trusted identity validation
→ canonical executor
→ profile + source link
→ STATE_DELTA: no_profile → profile_exists
→ RESULT + PROFILE_CARD + OPEN_CABINET
```

Telegram, Web and Unity then render the same action differently.

## Canonical Dynamic Interface Language

### A. Content blocks

- `TEXT`
- `QUOTE`
- `MEDIA`
- `ARTIFACT`
- `KNOWLEDGE_BLOCK`

### B. Navigation / structure blocks

- `BUTTON`
- `BUTTON_ROW`
- `MENU`
- `LIST`
- `CARD`
- `ENTITY_CARD`
- `PROFILE_CARD`
- `AVATAR_CARD`
- `SKILL_CARD`
- `MODE_CARD`
- `PAGE`
- `MODAL`
- `TREE`
- `WAY`
- `MAP`
- `WORLD`

### C. Process / state blocks

- `CHECK`
- `CHECKLIST`
- `SESSION`
- `STATE_DELTA`
- `STATUS`
- `PROGRESS`
- `RESULT`

### D. Action / economy blocks

- `ACTION`
- `POTENTIAL_ACTION`
- `OFFER`
- `ACCEPT`
- `TRANSACTION`
- `ACTIVATION`
- `CONNECT`
- `HANDOFF`

## Button passport

A canonical button/action should eventually carry semantic data independently from its visual phrase.

Conceptually:

```json
{
  "type": "button",
  "action": "PROFILE_CREATE",
  "target": null,
  "label_policy": "dynamic",
  "label_intent": "create profile",
  "fallback_label": "Создать профиль",
  "requires_confirmation": false
}
```

Possible `label_policy` values:
- `dynamic` — Orb/translator can phrase naturally;
- `localized_canonical` — deterministic locale dictionary;
- `server_canonical` — exact server-owned wording for payments, acceptance, irreversible actions or compliance-sensitive states.

This preserves the lively behavior the Telegram Orb already demonstrates without allowing wording freedom to change action semantics.

## Semantic UI AST principle

The flagship expresses meaning, not rendering implementation.

```json
{
  "blocks": [
    {
      "type": "text",
      "content": "Ты можешь создать профиль и продолжить уже как Актор Neo World."
    },
    {
      "type": "action",
      "action": "PROFILE_CREATE",
      "label_policy": "dynamic",
      "label_intent": "create my Neo World profile"
    }
  ]
}
```

The validator decides whether the action is allowed. Telegram may render an inline button, Web a card, Unity a spatial control.

## Progressive disclosure

```text
HORIZON
→ CURRENT INTEREST
→ RELEVANT CAPABILITY
→ OFFER IF EXPLICIT INTEREST
```

The same capability may therefore be represented as:
1. text-only horizon;
2. potential card;
3. detailed capability card;
4. offer + ACCEPT after explicit interest;
5. active action controls after activation.

## Model vs server responsibilities

### Flagship Orb may choose
- useful semantic block;
- relevant entity/capability/action;
- natural visible wording when label policy permits;
- desired handoff/action intent;
- how the UI continues the discourse.

### Server owns truth
- action identity;
- price;
- ownership;
- balance;
- dependencies;
- permissions;
- mode availability;
- connector state;
- action validity;
- irreversible state changes;
- transaction execution;
- canonical wording where exact wording is required.

## Channel adapters

### Telegram
Compact text + inline actions/cards; dynamic labels are already partially used; long-response handoff to Orb Web exists.

### Web
Rich cards, panels, modals, pages, lists, maps, media and live state.

### Unity / 3D
The same semantic blocks can become panels, objects, portals, paths, avatars and world interactions.

### VR / Metaverse
Actions may become world objects, portals, interactive surfaces and embodied navigation.

## Relationship with NEO-ENTITY-REGISTRY

The entity registry defines canonical objects such as PROFILE, AVATAR, MODAL, GAME, SPACE, 3D, VR and MV.

Dynamic Interface Language defines how Orb can **speak those objects into the current interaction**.

> Entity Registry: what kinds of things exist.
>
> Dynamic Interface Language: how Orb expresses, opens, offers, changes and acts on them now.

## Core design rule

> One semantic language, many renderers; stable actions, potentially living wording.
