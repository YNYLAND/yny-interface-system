# ORB DYNAMIC INTERFACE LANGUAGE

## Purpose

Dynamic Interface is the visual/action language of Orb.

The flagship model should not know how Telegram, Web, Unity or VR draws a button. It should express semantic interface intent. The server validates that intent against current state, rights, prices and available actions. Each channel adapter renders the same canonical intent in its native form.

Canonical pipeline:

```text
FLAGSHIP ORB
→ RESPONSE + UI INTENTS
→ POLICY / RIGHTS / LIVE STATE VALIDATOR
→ CANONICAL UI AST
→ CHANNEL ADAPTER
→ Telegram / Web / Unity / 3D / VR / future interface
```

## Current production reality

Today the language is fragmented.

### Model-visible UI in orb-api

The model may currently emit hidden `[UI]...[/UI]` with a tiny `buttons` array.

Model-generated actions currently accepted by `sanitizeOrbUi`:
- `open_skill_catalog`
- `open_skill` + `skill_key`
- `enter_ynychat` when owned and not already active
- `exit_ynychat` while YNY CHAT is active.

The model cannot directly create `purchase_skill`; BUY is generated server-side from a validated skill card.

### Server-generated UI

orb-api can already return:
- `buttons`
- `skill_cards`
- `modes`
- `catalog_mode_key`
- `list`
- state data.

Server actions already include:
- `get_list_state`
- `purchase_list_slots`
- `get_mode_state`
- `open_skill_catalog`
- `toggle_mode`
- `open_skill`
- `purchase_skill`
- `enter_ynychat`
- `exit_ynychat`.

### Telegram adapter

Telegram currently knows a different subset:
- `open_cabinet`
- legacy `connect_ynychat`
- `enter_ynychat`
- `exit_ynychat`
- `create_profile`
- Orb Web continuation button.

This mismatch is architectural debt: an Orb action can exist in the core but not have a Telegram renderer, or vice versa.

### Profile creation today

Telegram phrase `создай профиль` is intercepted before flagship cognition and directly executes profile creation. `create_profile` also exists as a Telegram callback concept, but is not part of the current orb-api model UI whitelist.

Target: profile creation becomes one canonical Orb action rather than a Telegram special case.

## Canonical language layers

### A. Content blocks

- `TEXT` — prose / explanation.
- `QUOTE` — cited or highlighted text.
- `MEDIA` — image, video, audio or preview.
- `ARTIFACT` — a created book, file, page, site, token, NFT, world, etc.
- `KNOWLEDGE_BLOCK` — reusable Infoteka/knowledge fragment.

### B. Navigation / structure blocks

- `BUTTON` — one concrete interaction.
- `BUTTON_ROW` — grouped immediate choices.
- `MENU` — navigation collection.
- `LIST` — ordered/unordered collection or Neo LIST slots.
- `CARD` — generic object summary.
- `ENTITY_CARD` — Neo World entity/passport summary.
- `PROFILE_CARD` — Actor/profile state.
- `AVATAR_CARD` — identity projection / avatar state.
- `SKILL_CARD` — capability/module state.
- `MODE_CARD` — SYSTEM / YNY CHAT / CORP or other environment mode.
- `PAGE` — full navigable surface.
- `MODAL` — temporary/embedded surface.
- `TREE` — hierarchy/branching view.
- `WAY` — path/roadmap view.
- `MAP` — geographic/spatial view.
- `WORLD` — 3D/VR/metaverse/world entry surface.

### C. Process / state blocks

- `CHECK` — one verifiable condition/result.
- `CHECKLIST` — current Gestalt/session checks.
- `SESSION` — semantic process state.
- `STATE_DELTA` — verified change from state A to B.
- `STATUS` — current state of an entity/action/process.
- `PROGRESS` — progress projection over checks/state deltas.
- `RESULT` — result produced by an action/session.

### D. Action / economy blocks

- `ACTION` — validated executable intent.
- `POTENTIAL_ACTION` — possible future action, not yet executable/selected.
- `OFFER` — commercial/activation proposition; only after offer gate.
- `ACCEPT` — Actor confirmation/acceptance.
- `TRANSACTION` — payment/transfer result/state.
- `ACTIVATION` — capability activation state.
- `CONNECT` — connection of an approved external account/service.
- `HANDOFF` — seamless move to another mode/surface while preserving context.

## Semantic UI AST principle

The flagship should express meaning, not rendering implementation.

Conceptual example:

```json
{
  "blocks": [
    {
      "type": "text",
      "content": "Ты уже определил роль. Следующий участок — публичное проявление."
    },
    {
      "type": "potential_action",
      "capability_key": "SMM_AUTOPILOT",
      "reason": "может закрыть distribution gap"
    }
  ]
}
```

This DOES NOT mean a purchase card is shown. Policy decides whether a potential action remains a sentence, becomes an info card, or becomes an offer.

## Progressive disclosure as rendering law

```text
HORIZON
→ CURRENT INTEREST
→ RELEVANT CAPABILITY
→ OFFER IF EXPLICIT INTEREST
```

Therefore the same capability may render differently:

1. distant horizon — mentioned in text only;
2. relevant potential — translucent/potential card or link;
3. explicit interest — detailed capability card;
4. activation requested — offer + price + ACCEPT;
5. activated — active capability/action controls.

## Model vs server responsibilities

### Flagship Orb may propose
- which semantic block is useful;
- which entity/capability/action is relevant;
- why it belongs in the current answer;
- desired handoff or action intent.

### Server must own truth
- prices;
- ownership;
- balance;
- dependencies;
- permissions;
- mode availability;
- connector state;
- action validity;
- transaction execution;
- irreversible state changes.

The model must never invent these values.

## Canonical profile-creation expression

Target:

```text
Actor: “создай профиль”
↓
Orb intent: ACTION / PROFILE_CREATE
↓
server validates trusted identity context
↓
executor creates profile and source link
↓
STATE_DELTA
  before: profile = absent
  after:  profile = exists
↓
response blocks:
  RESULT
  PROFILE_CARD
  BUTTON: OPEN_CABINET
```

Telegram may render this as an inline button, Web as a card, Unity as a spatial panel. It remains the same Orb sentence.

## Channel adapters

The canonical language is channel-independent.

### Telegram
Compact text + inline buttons/cards; long-response handoff to Orb Web.

### Web
Rich cards, panels, modals, pages, lists, maps, media and live state.

### Unity / 3D
The same entity/action blocks can become physical panels, objects, portals, paths, avatars and world interactions.

### VR / Metaverse
Cards/actions may be rendered as world objects, portals, interactive surfaces and embodied navigation.

## Relationship with NEO-ENTITY-REGISTRY

The interface registry defines canonical entities/forms such as PROFILE, AVATAR, MODAL, GAME, SPACE, 3D, VR and MV.

Dynamic Interface Language is how Orb can **speak those entities into the current interaction**.

The entity registry answers: “what kinds of interface/world objects exist?”

Dynamic Interface answers: “which of those objects should Orb express right now, in what state, and with which validated action?”

## Core design rule

> One semantic language, many renderers.

Orb should never have a separate mental UI vocabulary for Telegram, Web and Unity. The adapters translate one canonical response language into each environment.
