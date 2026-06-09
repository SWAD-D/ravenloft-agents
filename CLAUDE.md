# Native GPT Ravenloft — Claude Code Instructions

## Agent routing (automatic)

Before responding to any request, silently check whether it matches one of the agents below. If it does, invoke that agent instead of answering directly. Do not explain that you're routing — just do it.

| If the request is about… | Route to |
|--------------------------|----------|
| Starting a session, "where were we", "what's next", "catch me up" | `session-primer` |
| Adding a new NPC, character, voice, persona | `npc-adder` |
| Saving new state, schema change, versioning SaveSlot | `save-schema-extender` |
| Adding a zone action, location, scene category, investigatable thing | `scene-action-extender` |
| Adding a new DM tag, bracket tag, structured LLM output format | `metaparser-extender` |
| Auditing story beats, narrative gaps, beat coverage | `storybeat-auditor` |
| Combat bug, broken fight, wrong enemy stats, sprites wrong, wounds not saving | `combat-flow-debugger` |
| Background images, scene backgrounds, location art mismatch | `background-audit` |
| API costs, token usage, monthly spend, model pricing | `api-cost-estimator` |
| Tone, style, aesthetic, visual identity, fonts, colors, DALL-E prompts, writing voice | `tonality` |
| What to work on next, feature planning, sprint, roadmap, what's due | `dev-momentum` |
| Updating the roadmap, marking something complete | `roadmap-updater` |
| Run all analysis agents, full codebase audit | `orchestrator` |
| Chaining agents in sequence, running multiple agents back to back | `chain-runner` |

If a request spans multiple agents (e.g., "add a new NPC and update the roadmap"), invoke the `chain-runner` and tell it which agents to sequence.

If no agent matches, answer directly with full project context below.

---

## Project context

**What it is:** macOS SwiftUI Curse of Strahd RPG. AI DM via OpenAI streaming. ElevenLabs TTS per character. DALL-E backgrounds/portraits. Supabase cloud sync. SpriteKit combat.

**Source:** `Native GPT V1.4/` (~87 Swift files)
**Git branch:** `PolishWorkingV1.2`
**Roadmap:** `PRODUCTION_ROADMAP.md` — always read this first for current state
**Agent notes:** `AGENT_NOTES.md` — handoff file between Claude Code and GPT/Codex

**Central file:** `GameStore.swift` — @MainActor ObservableObject, ~1665 lines, owns all state
**Persistence:** `SaveManager.swift` — SaveSlot at v10, 25 stored fields, strict migration chain
**LLM routing:** `MetaParser.swift` — 11 regex patterns parse every DM response into game events
**NPC system:** `NPCRegistry.swift` — 13 profiles; adding one always requires 4 files
**Design system:** `Theme.swift` — Gothic.red, Gothic.dark, Gothic.ink, GothicButton

**Never commit:** `Secrets.swift` (gitignored — contains live API keys)
