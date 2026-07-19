# EcuadorCore Summary

EcuadorCore is a single-file interactive web experience (`ecuadorcore-final.html`) that answers "¿Qué es un Ecuador?" — a digital museum of Ecuadorian culture: expressions, viral internet moments, food, people, places, habits and facts. Built for Scarlet (@scarquez_). Tone everywhere: funny, proudly Ecuadorian, conversational, slightly exaggerated — never Wikipedia, never academic.

# Vision

The user should feel they are exploring Ecuador, never navigating a website. One continuous camera: Earth from space → cinematic flight → Ecuador becomes the hero. The homepage is a museum **lobby** (golden Ecuador silhouette at center, room bubbles orbiting it). Each bubble is the **entrance to a room** — its own exhibit with distinct visual language, pacing and interaction. Transitions must feel cinematic and fluid; there are no "pages", only camera movement and a circular veil that swallows the screen when entering a room.

# Current Architecture

World Map (3D Earth, Three.js) → CLICK AQUÍ → continuous Google-Earth-style zoom → Ecuador silhouette reveal → phrase ("el país de los ARRECHOS...") shown ONCE, then dissolves → Lobby (silhouette + 7 orbiting room bubbles + Shuffle/Sala sorpresa) → click bubble = veil transition into Room → click item = Content Card (overlay above room, room stays behind) → close card = back to Room → "← VOLVER A ECUADOR" = back to exact Lobby state.

Navigation state machine (single source of truth): `phase` = `space → zoom → hold → phrase → universe` (lobby) `→ room`. Cards/lightbox are overlays, not states. Escape key order: lightbox → card → room → (lobby).

# Current Rooms

All rooms defined in `CATS` (aliased `ROOMS`); items in `DATA`; layouts: `float` (Matter.js physics bubbles), `cards` (themed tile wall), `stickers` (checklist notes).

- **EN ECUADOR DECIMOS** 🗣️ — Ecuadorian expressions explained to foreigners. Float bubbles → editorial text card: big title, "🌍 PARA EL RESTO DEL MUNDO" (short translation), "🇪🇨 EN ECUADOR..." (playful cultural explanation), related chips. No images ever. 29 expressions, all content final/approved.
- **+593 CORE** 🇪🇨 — living archive of Ecuadorian internet: viral interviews, political phrases, memes, football. Float bubbles (short title only) → custom multimedia card (`CORE593` data). 7 entries, ALL with real local media. Videos autoplay on bubble click.
- **MADE IN ECU** ❤️ — notable Ecuadorians (hall-of-fame plaques on burgundy). Cards → media-tab modal. Placeholder content.
- **LA MEJOR GASTRONOMÍA DEL MUNDO** 🍽️ — dishes. Cards → media-tab modal. Placeholder content.
- **LUGARES QUE TIENES QUE CONOCER** 📍 — places (postcards on ocean blue). Cards → media-tab modal; every place already has a real interactive Google Maps embed (no API key). Photos uploaded but NOT wired yet.
- **FACTOS INTERESANTES** 🌚 — facts (glowing constellation cards on night purple). Cards → text modal. Placeholder content.
- **ECUATORIANO QUE SE RESPETA...** 😂 — playful checklist of habits, written as present-tense actions ("Come arroz con absolutamente todo."). Sticky-note stickers → short card (habit + one funny explanation via `d` field, no translation blocks). 20 habits, content final.

# Project Rules

- `ecuadorcore-final.html` is the ONLY production file. `ecuadorcore.html`, `ecuadorcore-fase1.html`, `ecuadorcore-opening.html` are historical iterations — never edit, never delete without asking.
- Never redesign the homepage, intro, silhouette, orbit, or navigation unless explicitly requested. Modify existing components instead of rebuilding. Owner sends incremental prompts; implement surgically.
- Preserve the museum model: rooms stay independent, each with its own theme; lobby must return to its exact state.
- Use uploaded media only. Never generate, invent, stock-source or fake media (especially political/viral content). Missing media = intentional styled placeholder (e.g. "VIDEO PRÓXIMAMENTE"); never broken image icons.
- Media filenames/paths must be ASCII: no accents, no spaces (macOS `file://` + NFD breaks them). Owner uploads folders with accented names; move/rename files into clean folders (`media/<id>/`), leave a LEEME.txt.
- Enner gallery (and all galleries) are APPEND-ONLY: new items go at the end, never replace.
- Bubbles are DOM divs synced each frame to Matter.js bodies via inline `transform`. NEVER put CSS transform animations/keyframes on `.bubble` (they override inline transforms and break positioning). Scale = `vis`/`targetScale` lerp in the engine loop; only opacity may be CSS-animated.
- One Matter.js engine total. Entering a room removes lobby door bodies from the world (`setLobbyDoorsVisible(false)`) and clears room pieces (`clearRoomPieces`) — keep this to avoid dupes/ghost collisions.
- `#roomHead` (back button) lives OUTSIDE `#room` on purpose (stacking-context fix). `#world` is `pointer-events:none` (bubbles are `auto`). Do not "clean up" either.
- Three.js is r128: use only r128 APIs (e.g. `Vector3.randomDirection()` does not exist — this once black-screened the whole app). Matter.js 0.19, both from cdnjs. Earth/cloud textures load from CDNs with fallbacks; app must survive offline/WebGL failure (`fallbackSpace`).
- Only use emojis that exist in Unicode (owner sometimes pastes invalid ones like 🫪 — substitute and tell her).
- Content language is Spanish (Ecuadorian voice). Owner-approved copy is verbatim — including line breaks (`<br>`) — don't rewrite it. Copy you draft yourself: flag it for her review.

# Folder Organization

Everything lives in one folder (`ECUADOR CORE/`): the four HTML files at root (all code — HTML/CSS/JS — is inline in each single file; no build, no dependencies beyond CDN scripts; opened directly via `file://`). All media under `media/<entryId>/` (e.g. `media/enner/`, `media/amen/`, `media/viudas/`) referenced by relative paths from the data objects; each folder has a LEEME.txt with naming rules. `media/lugares que tienes que conocer/` holds 15 unwired place photos (accented filenames — must be renamed before wiring). `AI_HANDOFF.md` = this file.

# Current Project Status

- ✅ Cinematic intro: rotating 3D Earth, continuous zoom, silhouette reveal, phrase dissolve.
- ✅ Lobby: organic orbit around silhouette (hover slows, drag throws + returns), Shuffle (reorders orbit), "+ Más Ecuador" (random room).
- ✅ Museum navigation: veil transitions, 3-state model, isolation + freeze bugs fixed, re-entry safe.
- ✅ 7 themed rooms with 3 layout engines; text/media/CORE593 card systems with per-item overrides.
- ✅ Content final: DECIMOS (29), ECUATORIANO QUE SE RESPETA (20), +593 CORE (7 entries, all real media: 5 autoplay videos, 1 image, 2-meme zoomable swipe gallery).
- ⏳ Pending: wire LUGARES photos (already in folder, need ASCII rename + gallery arrays); real content for GASTRONOMÍA, MADE IN ECU, FACTOS (currently emoji-title placeholders with auto-placeholder cards).
- ⏳ Pending: "contribution" feature mentioned by owner (user-submitted content) — never specified, not built.
- ⚠️ Limitation: Earth textures/CDN scripts need internet on first load; final zoom masks texture pixelation with silhouette + blur by design.
- ⚠️ Limitation: lightbox zoom is basic (tap-to-toggle 2x, scroll-pan); no pinch gestures.
- ⚠️ Not systematically tested on mobile/touch devices.

# Before Making Changes

1. Read the target section of `ecuadorcore-final.html` first — the code is the source of truth, not this doc.
2. Confirm which room/feature the request touches; touch nothing else.
3. Check whether the request conflicts with a Project Rule; if so, ask before proceeding.
4. New content goes in the data objects (`CATS`, `DATA`, `CORE593`) — never hardcode content into markup/renderers.
5. New media: verify the file exists, move/rename to ASCII path under `media/<id>/`, then wire.
6. Preserve exact owner copy; mark any copy you invent for her review.
7. After editing: syntax-check the inline JS (extract + `node --check`) and sanity-check data (no duplicate titles, valid relatedIds, files exist).
8. Test the full loop mentally or in browser: intro → lobby → each changed room → card → back → lobby interactive again.
9. Never leave dead code paths, duplicate physics engines, or listeners that survive room exit.
10. Keep everything in the single file; no new files/frameworks/build steps unless she asks.
