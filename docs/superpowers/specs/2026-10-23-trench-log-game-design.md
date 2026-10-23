# Trench Log Game — Design

Status: draft for Edson's review. Date: 2026-10-23.

## Goal

Build a standalone, two-player online action RPG in the style of Secret of Mana. Players do not need to have read the book. The book is inspiration only. The book's Part II and the game share one story, written weekly from real engineering work.

## Decisions already made

- Platform: browser, hosted on GitHub Pages. Engine: Godot 4, web export.
- Players: exactly 2, in different places, joining by a short room code.
- World: fantasy with sleeping machines under the ruins (a mix of fantasy and the book's AI themes).
- Book: Part I (chapters 1–9, São Paulo 2031) stays as written. Part II starts at chapter 10 and tells the game's story in the game's world.
- Cast: based on the real agents (see Cast).
- Free and open source tools first.

## Scope of the first version (the "slice")

One town, one dungeon, one boss, one finished story arc. Both heroes playable. Ring menu working. About 30 minutes of play. Nothing outside this slice is in scope for version one.

## Game design

Core feel, taken from Secret of Mana:
- Real-time combat in a top-down view.
- A charge meter limits how often you can attack at full power.
- A ring menu for items, spells and weapons.
- Weapons level up with use.
- A follower: if the second player is absent, an AI plays that hero.

## Architecture

Three parts, each with one job.

1. **Game engine (`game/`)**. Godot project. Holds the rules: movement, combat, damage, leveling, quest state, the ring menu. It contains no story text.
2. **Episode packs (`episodes/`)**. Plain data files (JSON plus map files). Each pack holds quests, dialogue, a map, enemies and a boss. The engine loads a pack at start. A new pack needs no engine rebuild.
3. **Canon (`canon/`)**. The facts of the story. Both the book and the episode packs are written from it.

### Two-player networking

- One player creates a room and gets a short code. The other enters the code.
- Browsers send game data directly to each other (WebRTC).
- A small matchmaker server helps the two find each other. It stores no player data and is only used to connect them. It runs on a free tier (Cloudflare Workers). If that proves unsuitable, the fallback is a small WebSocket server on another free host.
- The host's game is the referee for enemies, loot and quest state. The guest sends inputs and receives state. This keeps both screens in step.
- Godot's web export must run without special server headers, because GitHub Pages cannot set them. Use the single-thread web export.

### Weekly content flow

1. The weekly run reads the canon and the real events of the week (merges, failed checks, releases).
2. It maps events to story beats using `canon/agent-map.md`.
3. It writes the book chapter and the matching episode pack.
4. The continuity check reviews both (see below). On APPROVE, both are published. On BLOCK, the run rewrites once. If it still fails, it stops and tells Edson.
5. The run updates the canon.

The weekly run writes data only. It never writes game code.

## Canon and continuity

Files in `canon/`:
- `characters/<name>.md`: voice, traits, relationships, secrets, current goal, and a log of what has happened to the character.
- `world.md`: rules of the world, the map, and what magic and machines can and cannot do.
- `timeline.md`: events in order, each with who knows about it.
- `agent-map.md`: which real event becomes which story beat.
- `recap.md`: a one-page summary of the last three chapters, rewritten each week and added to every prompt.

Continuity check: a second agent reads each draft against the canon and answers APPROVE or BLOCK. It checks for contradictions, dialogue out of voice, characters appearing after they left the story, and characters knowing things they should not.

## Cast

Based on the real agents, so real events can drive the story.

- **Louis**: a watcher spirit. Calm, speaks in warnings.
- **The pr-reviewer**: a gatekeeper who judges offerings at the gate. Says APPROVE or BLOCK.
- **Hermes and the Gemini council**: rival and allied voices that argue about what to do.
- **Workers**: small helper constructs that go off to build things and come back.
- **Kael**: playable hero one, carried over from the book.
- **Hero two**: an original character, playable. Name and design are decided in the canon step (see Open decisions).

## Testing

- Automated tests for pure rules: damage, leveling, quest state. Written test-first.
- An automated check that loads every episode pack and confirms quests, maps and dialogue are valid. It runs in CI.
- Multiplayer is checked by hand with two browsers before each release.
- CI cost: tests only, no macOS runners, kept within the existing GitHub Actions budget rule.

## Art and audio

Free packs first (CC0 pixel art, for example from Kenney). Credits go in the repo. Custom art can replace them later.

## Risks

- **Voice drift** across weeks. Mitigation: voice notes per character and the continuity check.
- **Browser networking limits.** Some home networks block direct WebRTC. Mitigation: use a free relay (TURN) for those cases, and test early.
- **Scope growth.** Mitigation: the slice is fixed. New features wait for version two.

## Open decisions for Edson

1. Hero two: name and look. Default if you have no preference: I propose three candidates in the canon step and you pick one.
2. Repo layout: keep the game in this repo (`game/`, `episodes/`, `canon/`) or move it to a new repo. Default: keep it here.
3. Book cron: it changes only after `canon/` exists. The new prompt will read the canon, write chapter and episode, run the continuity check, and update the canon.

## Next step

After you approve this spec, write an implementation plan and build the slice in this order: canon files, engine rules with tests, one episode pack, networking, then the book cron change.
