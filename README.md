# Ravenloft Agents

Claude Code agent toolkit for the [Native GPT Ravenloft](https://github.com/SWAD-D/native-gpt-ravenloft) project — a macOS SwiftUI D&D RPG (Curse of Strahd) with an AI Dungeon Master, per-character TTS, DALL-E art, and Supabase cloud sync.

These agents give Claude Code deep, persistent knowledge of the codebase so you never have to re-explain context at the start of a session.

---

## Setup

Copy the `agents/` folder into your project's `.claude/` directory:

```bash
cp -r agents/ /path/to/native-gpt-ravenloft/.claude/agents/
```

Claude Code picks them up automatically. Invoke any agent by name in conversation.

---

## Agents

### Analysis Pipeline

Run these to build a ground-truth picture of the codebase. Run `orchestrator` to chain all three in one shot.

| Agent | Trigger | What it does |
|-------|---------|--------------|
| `orchestrator` | "run the orchestrator" | Chains codebase-reader → dependency-analyst → skill-architect with no input. Writes outputs to `.claude/analysis/`. |
| `codebase-reader` | "use the codebase-reader" | Reads all 87 Swift files line by line. Produces a structured catalog: role, key types, published state, key methods, outbound deps, notable patterns. Ends with a layer map. |
| `dependency-analyst` | "use the dependency-analyst" | Traces ownership tree, data flows for 7 major user actions, @Published propagation, SaveSlot schema, all 15 AI call sites, coupling hotspots, concurrency map. |
| `skill-architect` | "use the skill-architect" | Designs the full agent toolkit from the catalog and dependency map. Produces full specs with priorities. |

---

### Session Workflow

| Agent | Trigger | What it does |
|-------|---------|--------------|
| `session-primer` | "start my session" | Reads `PRODUCTION_ROADMAP.md`, last 20 git commits, and `AGENT_NOTES.md`. Returns a ≤300-word briefing: current state, last done, what's next, open questions, active risks. |
| `roadmap-updater` | "update the roadmap" | Reads git diff and log, then updates `PRODUCTION_ROADMAP.md` — moves completed items to ✅, rewrites current state, identifies what's now unblocked. |

---

### Feature Agents

| Agent | Trigger | What it does |
|-------|---------|--------------|
| `npc-adder` | "add a new NPC" | Interviews you (name, personality, voice, Strahd relationship, knowledge domains) then produces exact compilable Swift diffs for `NPCRegistry.swift`, `SpeakerMap.swift`, `NarratorEngine.swift`, and `NPCPortraitStore.swift`. |
| `save-schema-extender` | "I need to persist [X] state" | Knows SaveSlot is at v10 with 25 stored fields. Produces the new field additions, version bump, migration case in `init(from:)`, and `makeCurrentSlot` patch. Warns on Supabase JSON impact. |
| `scene-action-extender` | "add a zone action" | Validates the existing SceneExplorer schema first (action counts, duplicate IDs, missing `isDocumentAction` flags), then adds the new location or action across `SceneExplorer.swift`, `SceneBackground.swift`, and `BackgroundManager.swift`. |
| `metaparser-extender` | "add a new DM tag" | Adds a new structured tag to MetaParser safely: validates the regex against sample LLM output before writing, enforces the `private enum RX` pattern, updates `MetaParseResult`, and documents the tag format for DM prompts. |

---

### Audit & Debug Agents

| Agent | Trigger | What it does |
|-------|---------|--------------|
| `storybeat-auditor` | "audit the story beats" | Cross-references 50+ `StoryBeat` cases against `NPCRegistry` (13 profiles), `Lore.swift`, and `SceneExplorer`. Finds unreachable beats, knowledge domain gaps, beats without trigger conditions, and orphaned references. |
| `combat-flow-debugger` | "combat is broken — [symptom]" | Routes the symptom through a 8-layer combat stack (MetaParser trigger → CombatManager → SpriteKit scenes → CombatHUD → outcome reporting). Returns layer ownership, ranked candidate bug sites with confidence ratings, and diagnostic steps. |
| `background-audit` | "audit the backgrounds" | Cross-checks `SceneBackground` (54 cases) vs `BaroviaLocation` (18 cases) vs `BackgroundClassifier` enum vs cache keys. A cache key mismatch means a background regenerates on every visit — this catches that before it costs API credits. |
| `api-cost-estimator` | "estimate my API costs" | Reads all 15 AI call sites across OpenAI (DM narration, NPC dialogue, story beats, document generation, image generation) and ElevenLabs (TTS, SFX). Returns a cost breakdown table with session totals, monthly estimates, and caching opportunities. |

---

## Architecture context

The codebase has 87 Swift files across 10 layers. Key facts every agent knows:

- **`GameStore.swift`** is the single orchestrator — `@MainActor final class`, ~1665 lines, owns all `@Published` state, routes every major operation
- **`SaveSlot`** in `SaveManager.swift` is at schema v10 with 25 stored fields and a v1–v10 migration chain — never bump the version without a migration case
- **`MetaParser.swift`** drives every game event from LLM output via 11 compiled regexes — highest-risk file to change casually
- **15 distinct AI call sites** — 7 bypass `OpenAIClient` entirely and call OpenAI via raw `URLSession` (StoryMaster, NPCEngine, NPCMemorySummarizer, BackgroundClassifier, InsightEngine, DocumentGenerator, BackgroundGenerator)
- **`NPCRegistry.swift`** has 13 named NPC profiles; adding one requires touching 4 files consistently
- **`SceneExplorer.swift`** has 10 `SceneCategory` cases × 5–6 `ZoneAction`s each; `isDocumentAction = true` flags actions that trigger `DocumentGenerator`

---

## Project links

- Game repo: [SWAD-D/native-gpt-ravenloft](https://github.com/SWAD-D/native-gpt-ravenloft)
- Stack: SwiftUI · macOS · OpenAI · ElevenLabs · DALL-E · Supabase · SpriteKit
